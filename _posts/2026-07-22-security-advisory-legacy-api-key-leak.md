---
title: "Security advisory: Possible leak of legacy API keys via improper cache configuration"
layout: post
date: 2026-07-22 21:02:00 +0000
author: Colby Swandale
author_email: colby@rubygems.org
---

A CDN caching bug on RubyGems.org could hand one account's API key to another person for up to an hour. If you signed in to RubyGems.org with a gem client older than v3.2.0, your key could have been exposed (the technical details are below). Currently, 18% of sign-ins through `gem signin` come from an affected version, and for the first several years of this bug, before we changed the client's sign-in path in December 2020, it was every gem client.

We've reviewed the access logs we keep and found no sign of a legacy key being used maliciously, and no user or support report has suggested otherwise. Those logs only cover a recent window, though, a small slice of the years this bug is estimated to have been present. Rather than lean on log analysis to rule out any potential abuse, we've revoked every RubyGems.org legacy key, and we're asking owners to check their own gems.

Your already-published gems were never at risk: existing releases cannot be rewritten, and installing gems was never affected. What was at risk is the key itself. Anyone holding a leaked key could act as that account: publish a new gem version, yank versions, add themselves as an owner.

If you had MFA enabled for API requests, your key could still have leaked, but MFA blocks a leaked key from pushing, yanking, or changing owners.

## What account holders should check

Existing releases cannot be rewritten, so if you're an account owner, what you are checking for is anything added or removed under your name. On each of your gems, look for versions you did not publish (especially one higher than your latest), unexpected yanks, unfamiliar owners or maintainers, trusted publishers you did not configure, and webhooks you do not recognise.

We're notifying users whose account held a legacy API key at any point, since those are the accounts this bug could have exposed, so please keep an eye on your inbox. You can immediately review your account's [API Key history](https://rubygems.org/profile/api_keys) through your profile page.

## What is a RubyGems.org Legacy API Key

A Legacy API key is a single credential that carries every gem-management permission at once. It isn't tied to a single gem, so those permissions apply across every gem you own, and it has no expiry, so it stays valid until you delete it.

In practice that means the key can do many operations the API exposes for publishing and maintaining your gems: push a new version, yank an existing one, add or remove owners, change webhooks, and configure trusted publishers. What it can't do is act as a full account login. No API key can change your password or email, alter your MFA settings, or delete your account.

The word "legacy" undersells how ordinary these keys were. When this endpoint was built, and for years afterward, this was simply the API key: the only kind RubyGems.org issued, and the only way the gem CLI could sign you in. Scoped keys arrived in RubyGems v3.2.0 in December 2020, and it's at that point the older keys became "legacy". So for most of the roughly nine years this bug existed, the affected credential wasn't a deprecated relic that a few stragglers still used. It was the standard way everyone authenticated.

## Who is affected

* Users of the rubygems client older than v3.2.0, which signs in by calling `GET /api/v1/api_key`. Notably, this includes the vendored rubygems with current macOS (Tahoe), version `3.0.3.1` at `/usr/bin/gem`, so the affected client is a standard system tool, not an edge case.
* Recipients of a leaked key, who may be entirely uninvolved account holders. A cached hit wrote another account's key into their local credentials file, and from that point they acted as the other account until they signed in again.

## Technical detail

Under a specific interaction between the response compression and cache headers, our CDN cached the successful response and served the same freshly created key to subsequent callers on the same edge node (POPs) for up to an hour, without re-checking their credentials.

The result was that one account's API key could be handed to another party, with no attacker involved. Once one user's sign-in response was cached at a Fastly edge node, the next user signing in through that same node within the hour received the earlier user's key instead of a fresh one of their own. And because cache hits were served at the edge without contacting the origin, an unauthenticated party could also poll the endpoint to harvest whatever key happened to be cached.

The chain is as follows:

1. `GET /api/v1/api_key` authenticates with HTTP Basic and creates a new legacy key, returning it in a `200` body.
2. The Ruby client sends `Accept-Encoding: gzip` by default (via `Net::HTTP`). `Rack::Deflater` replaces the response body with a `GzipStream`.
3. `Rack::ETag` can't read the gzipped body, so it falls back to a bare `Cache-Control: no-cache`, with no `private` and no `Set-Cookie`.
4. With only a bare `no-cache` and no `Vary: Authorization`, Fastly cached the `200` for up to an hour under a shared cache key for the single CDN node that processed that request. Every cache hit during that window returned the same key, served at the edge without re-authenticating the caller.
5. A successful response here is itself the API key.

A detail worth calling out, because it explains why casual testing missed this: the bug is conditional on `Accept-Encoding: gzip`. A plain `curl` request, which sends no such header, received a correct-looking `Cache-Control: private, must-revalidate` response and did not reproduce the issue. The vulnerable path was the one the real Ruby client exercises by default.

The application-side trigger (`Rack::Deflater`) was introduced on 10 October 2016 (commit `03d89c0`). The date the response first became edge-cacheable cannot be established from the repository, so the conservative assumption is that the endpoint was exploitable for most of the roughly nine years since.

**We have assigned this incident the following CVSS score:**

| CVSS Version | 4.0 |
| :---- | :---- |
| Base Score (CVSS-B) | 7.3 (High) |
| Environmental Score (CVSS-BE) | 7.2 (High) |
| **Overall Score** | **7.2 (High)** |
| Macro Vector | 111100 |

**Vector String:**

`CVSS:4.0/AV:N/AC:L/AT:P/PR:N/UI:P/VC:L/VI:H/VA:H/SC:L/SI:H/SA:H/CR:L/IR:H/AR:H/MAV:N/MAC:L/MAT:P/MPR:N/MUI:P/MVC:L/MVI:H/MVA:H/MSC:L/MSI:H/MSA:H`

## Impact and its limits

Adding an owner and registering a trusted publisher persists even after the key itself is revoked, which is why key revocation alone is necessary but not sufficient, and why we asked owners to review their ownership and trusted-publisher settings.

There are firm limits on what was possible:

* Existing gem releases cannot be altered by users. Any existing name, version, and platform is immutable; pushing an already published version is rejected with HTTP 409 "Repushing of gem versions is not allowed". A yanked version number cannot be reused either.
* What "clobbering" actually means here is shipping a new, higher version that supersedes the current one as the default install, yanking versions, or (after yanking all versions) reclaiming a name. A new platform variant of an existing version number is a distinct, allowed artifact rather than an overwrite. None of this rewrites an already-published release.
* MFA helps when enabled on API requests. A leaked key is neutralised for push, yank, owner changes, and trusted-publisher changes when the account has enabled it for API requests (`ui_and_api`), which forces multifactor authentication on those actions. Accounts set to disabled, or `ui_and_gem_signin`, get no such protection against a leaked key.

## When the vulnerable client path was retired

The `gem signin` flow stopped calling `GET /api/v1/api_key` in RubyGems v3.2.0, released 10 December 2020, which switched sign-in to `POST /api/v1/api_key` as part of the [introduction of scoped API keys](https://github.com/ruby/rubygems/pull/3840).

That client change did not remove the risk, because the `GET` endpoint was deliberately kept on the server so that actively maintained clients would keep working. It continued to mint new API keys for every caller until the fix below. In other words, the client version determined only whether the modern `gem` reached the endpoint; the endpoint itself remained live and cacheable for everyone, including direct API callers, for the whole period.

## Why we didn't detect this before

Attribution of any misuse is difficult by design of the platform, and we want to be plain about that rather than reassuring:

* Every action taken with a key is recorded under the legitimate key-holder. The pusher, the event actor, the notification emails, and the rate-limit bucket all read as the rightful owner.
* The only signals that distinguish a different caller are the source IP address and user-agent. There was no notification on key use (only on key creation), and no new-IP or velocity alerting on API keys.
* As noted above, the flaw only manifested for gzip-compressed requests, so header inspection with ordinary tools did not surface it.

The worst-case exposure dates back to 2016 and our log history is limited. As a result, most of that window cannot be reconstructed. This is why we treated revocation as the remedy and any log review as best-effort corroboration only.

## The fix

Deployed in commit `d3d11c0` (9 July 2026):

* The authentication response now sets `Cache-Control: private, no-store`, `Surrogate-Control: max-age=0`, and appends `Authorization` to `Vary` on the API key response and on the other authenticated API endpoints. The sign-in response can no longer be stored in a shared cache, and any cache that does store a response now varies on the credential.
* We purged the affected objects from Fastly before revoking keys, so re-issued keys could not be re-poisoned.

## Remediation and what breaks

We've revoked all legacy API keys. No other keys were affected. Scoped keys created through the RubyGems.org UI and the RubyGems CLI were never exposed by this bug, which only affected the legacy sign-in method, not the way scoped keys are created. Short-lived OIDC and trusted-publisher keys were also left alone, since they come from a separate token exchange, expire on their own within minutes, and can't leak through this path.

Because only legacy keys were revoked, the breakage is limited and predictable:

* Both `gem install` and `bundle install` are anonymous and keep working.
* The next `gem push`, `gem yank`, or `gem owner` using an expired legacy key will return an HTTP 401 response. To recover, create a new API key at [https://rubygems.org/profile/api_keys](https://rubygems.org/profile/api_keys) and load it into your local RubyGems configuration by following the [API key guide](https://guides.rubygems.org/api-key-scopes/).
* For CI or automation using legacy keys, the next gem push will return an HTTP 401 response until you replace the stored key (for example `RUBYGEMS_API_KEY` or `GEM_HOST_API_KEY`) with a newly created one. CI that only installs gems is unaffected, and CI already using trusted publishing (OIDC) keeps working, since those keys weren't revoked and are re-minted on each run.
* If you moved off legacy keys to scoped keys, nothing was revoked and no further action is required.

We've also retired the old sign-in endpoint (`GET /api/v1/api_key`), so `gem signin` on RubyGems clients older than v3.2.0 no longer works. Creating a key on the website and loading it as above works on any client version, and replacing your stored key this way also clears any key you may have been unknowingly holding for another account.

We recommend scoped keys over full-access keys, MFA enabled for API access, and trusted publishing over long-lived keys in CI.

## Timeline

* 10 October 2016: `Rack::Deflater` added, introducing the application-side trigger (commit `03d89c0`).
* 10 December 2020: RubyGems v3.2.0 released; the modern `gem signin` stops calling the `GET` endpoint, though the endpoint remains for older clients.
* 6 July 2026: reported to RubyGems.org by Luke Marshall
* 9 July 2026: root-cause fix deployed (commit `d3d11c0`), Fastly purged
* 23 July 2026: Legacy API keys revoked, affected users notified & public disclosure published

## Credit

Thanks to Luke Marshall from [trufflesec.com](https://trufflesec.com) for the responsible report and for working with us through remediation. This security report has been published as [GHSA-9j48-x3c3-mrp2](https://github.com/rubygems/rubygems.org/security/advisories/GHSA-9j48-x3c3-mrp2)

## Our commitment to security

I lead the team that maintains RubyGems.org and my name is on Bundler, rubygems & RubyGems.org going back almost a decade. When the report came in, the first thing I felt wasn't so much about the bug itself. It was the question of how it sat there, in the open, for the better part of a decade, and we never caught it ourselves. We found this because someone told us, not because we saw it, and I want to be honest about that.

Beyond the actions we've taken today, we're looking at expanding the two-factor authentication requirement across the platform in 2026, so a single leaked key is worth much less on its own. If you haven't already, please turn it on today and set it to apply to both the [UI and API](https://guides.rubygems.org/setting-up-multifactor-authentication/).

None of this makes RubyGems.org 100% safe, and I won't pretend it does. Each of these fixes a specific weakness this incident exposed, and we'll keep working on the rest. RubyGems.org is infrastructure that nearly every Ruby developer depends on, and keeping it secure is the core of what our team is here to do.

Colby Swandale, RubyGems.org Technical Lead
