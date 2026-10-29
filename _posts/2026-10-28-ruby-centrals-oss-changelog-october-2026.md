---
title: "Ruby Central’s OSS Changelog: October 2026"
layout: post
author: Kirk Haines
author_email: kirk@rubycentral.org
---

*Originally published on [Ruby Central](https://rubycentral.org/news/ruby-centrals-oss-changelog-october-2026/).*

## Relaunching the changelog

It has been a long time since the last Ruby Central Changelog, and we’re bringing it back. Ruby Central’s open source team operates [RubyGems.org](https://rubygems.org/), a service many of us rely on every day, and supports the packaging ecosystem around it. The Changelog is where we will share that work. Each issue will cover what is changing, why it matters, and where you can get involved.

This is the first issue edited by our new Community Manager, Kirk Haines, who joined Ruby Central in August. If you missed his introduction in the Ruby Central August [README](https://mailchi.mp/cf35f86b5ae4/august-2026-readme-newsletter), the short version is that Kirk has been using Ruby since 2001 and is glad to give back to the language and the community that fueled so much of his career.

## Improving operations of RubyGems.org

Over the past year, the team has invested in the operational side of RubyGems.org. This work included how the service is monitored, how incidents are handled, and how abuse gets cleaned up.

The team has reorganized the on-call rotation, which follows the sun so that someone awake is responsible for RubyGems.org at every hour of the day. Runbooks for monitoring and common operational tasks have grown from nine to 72, so whoever is on call has a procedure to work from, which makes operational knowledge easier to share across the team. Offboarding has a checklist, every incident gets a written postmortem, and expanded logging and monitoring help the team detect and investigate problems. Much of this came out of Ruby Shield, our four-year partnership with Shopify. Its [retrospective](https://rubycentral.org/news/ruby-shield-a-reflection-on-four-years-of-securing-the-ruby-supply-chain/) tells the full story.

The team also built better tools for cleaning up abuse. Admins are now able to more easily manage the day-to-day operations and respond faster to incidents as they emerge. That tooling is what we used in the August typosquat response.

Under the hood, we’ve kept RubyGems.org current with the ecosystem it serves. It now runs on Ruby 4 and Rails 8.1, and we regularly upgrade our core infrastructure to stay up to date on the latest patches. Our next piece of infrastructure work is upgrading our database to Postgres 18.

## Keeping RubyGems.org secure

RubyGems.org serves over 1 billion requests a day, and keeping it available and trustworthy was a large part of this year’s work. In early April, bots from many sources spent a week scraping data about every published gem, and the volume disrupted parts of the site. We [temporarily disabled language switching]({% post_url 2026-04-07-temporarily-disabling-language-support %}) to keep the site stable while we reworked how those pages were cached.

Two other changes closed off less visible risks. Aaron Patterson audited all 180,000+ gems published on RubyGems.org to build a schema from real gem metadata, and [gem pushes are now validated against it]({% post_url 2026-04-09-protecting-rubygems-from-the-outside-in %}). The validator extracts only the values it expects from a gem’s metadata instead of letting Ruby build objects from it, which closes off an entire class of attacks on the push endpoint. RubyGems.org also now checks passwords against Have I Been Pwned at login, registration, and password reset.

In July, Luke Marshall of Truffle Security reported that responses from our [legacy API key endpoint could be cached by our CDN]({% post_url 2026-07-22-security-advisory-legacy-api-key-leak %}) and served to the wrong person for up to an hour. We deployed a fix three days later, retired the old endpoint, revoked every legacy API key, and notified everyone whose account had ever held one. Our access logs show no sign of a legacy key being used maliciously. If a `gem push`, `gem yank`, or `gem owner` from an older setup now returns a 401, create a new API key on RubyGems.org and update your local and CI configuration.

Some of this work reaches beyond the service itself. In April we [started using AI models to look for vulnerabilities]({% post_url 2026-04-29-scaling-rubys-defenses-with-ai %}) in the gems the Ruby ecosystem depends on most, prioritized by the OpenSSF Criticality Score. With funding from Alpha-Omega, that work has grown into our Security Engineers in Residence program. The Gem Security team now scans top gems and works with their maintainers to fix what they find, with 90 days as the upper limit for coordinated disclosure.

The 619 gems on our list have been downloaded a combined 158 billion times, and 26 of them have each passed a billion downloads. About 317 of those 619 have been scanned so far. A person reviews every finding, and those which survive that review are reported to the relevant maintainer. Of the 134 findings our reviewers evaluated, 120 (90%) were rejected because they weren’t reachable by an attacker, described intended behavior, or duplicated another report. About 15 have been reported to maintainers or advisory channels, and six security advisories have been opened. Some of those are still under embargo, and we’ll share more as the fixes ship.

One that is [now public is in rubyntlm](https://github.com/WinRb/rubyntlm/security/advisories/GHSA-cx24-f932-rr55), an NTLM authentication gem with over 94 million downloads. Its NTLMv2 password check ignored the password it was given and checked every login against an empty password instead, so an application relying on it would let in anyone who knew a valid username. The maintainer rated it critical, 9.3 out of 10, and released a fix in rubyntlm 0.6.6. If you use [rubyntlm](https://rubygems.org/gems/rubyntlm), upgrade to 0.6.6 or later.

If you maintain a gem and would like it reviewed, or you’ve found a vulnerability in a gem and want to report it privately, reach out to the Gem Security team at [gem-security@rubygems.org](mailto:gem-security@rubygems.org).

## Making the work sustainable

In [Software Foundation Musings](https://rubycentral.org/news/software-foundation-musings/), Marty Haught, Ruby Central’s Director of Open Source, looks at what Ruby Central needs to become a stronger software foundation. The main pieces are clear governance, policies and documentation that support the work, and recurring funding that can sustain it. He is learning from other foundations and considering what will fit the Ruby ecosystem. This is a work in progress on the path to a more solid organizational plan.

We think infrastructure should be boring in the sense that you can count on it. When you install a gem or deploy an application, your reasonable expectation is that everything just works. Making that possible takes ongoing maintenance and a team that can keep doing it. The funding and organizational work are part of that.

## Announcing the RubyGems teams

We’re grateful to the previous maintainers of RubyGems, Bundler, and RubyGems.org for the years of work they devoted to building, maintaining, and improving these projects. Their expertise and care continue to benefit everyone who depends on Ruby’s packaging ecosystem. The work described in this Changelog builds on their contributions.

Over the past year, Colby Swandale, Hiroshi Shibata, and Marty Haught have expanded the teams supporting RubyGems, Bundler, and [RubyGems.org](https://rubygems.org/). Members come from Ruby Core, Ruby Central, the volunteer community, and companies that contribute engineering time, so responsibility for these critical tools is widely shared. We’re excited to introduce them.

### RubyGems core team

With the RubyGems and Bundler repositories now part of Ruby Core, the clients have a dedicated core team. It’s led by Hiroshi Shibata, the project’s Ruby Core contributor since 2014, and includes Kouhei Sutou, Colby Swandale, Aaron Patterson, Edouard Chin, Eileen Alayce, and Mike Dalessio.

### RubyGems security team

Security for both the clients and RubyGems.org has a dedicated team composed of Hiroshi Shibata, Colby Swandale, Dushan Karovich-Wynne, Maciej Mensfeld, Jenny Shen, Patrick Linnane, Mike Dalessio, and Aaron Patterson.

### RubyGems.org service teams

The [RubyGems.org](https://rubygems.org/) service has multiple teams that oversee maintenance, on-call, and security.

Maintenance is handled by Colby Swandale, Dushan Karovich-Wynne, Jenny Shen, and Issy Long. They are joined by in-kind engineers on rotation. The current engineers in the rotation are Brian Tenggren from [thoughtbot](https://thoughtbot.com/) and Anton Bondaronok from [Rubyroid Labs](https://rubyroidlabs.com/).

Our on-call team has a primary and a secondary rotation. Our primary rotation includes Colby Swandale, Jenny Shen, Mike Dalessio, and Arun Agrawal. We have a secondary rotation of Evan Phoenix, David Radcliffe, and Joshua Young.

We have a specialized security team, Gem Security, who are focused on running our [Security Engineers in Residence](https://rubygems.org/pages/security-engineers-in-residence-faq) program using AI models to proactively scan top gems for vulnerabilities. Its members are Dushan Karovich-Wynne, Matt Mongeau, Patrick Linnane, and Colby Swandale.

## RubyGems news

### RubyGems and Bundler 4.1.0.beta2

The second beta of the 4.1 line [went out on October 8th]({% post_url 2026-10-08-4.1.0.beta2-released %}), following [beta1 on September 9th]({% post_url 2026-09-09-4.1.0.beta1-released %}). Dependency resolution is faster, and when a Gemfile cannot be satisfied the error now tells you which constraints actually conflict instead of dumping the whole graph. Installs reuse a gem cache shared between RubyGems and Bundler, so the same gem is downloaded once for all your projects. Your RubyGems.org credentials can move out of a plain text file and into the operating system’s credential store, and `gem push` attaches provenance to your releases with no extra setup. Gemfiles gain `override`, which rewrites the version requirements on a gem and, new in beta2, drops or replaces one dependency of a gem. A new `prune` setting strips caches and git metadata from your bundle path. Please try it with `gem update --system --pre` before the final release.

Read more: [What’s New in RubyGems/Bundler 4.1, Part 1: override and prune](https://dev.to/hsbt/whats-new-in-rubygemsbundler-41-part-1-override-and-prune-2ngn)

### Dependency cooldowns

Most supply chain attacks against RubyGems exploit a narrow window. An account is compromised, a malicious version ships, and any `bundle install` in the minutes that follow resolves straight to it. Dependency cooldowns, which shipped as part of Bundler 4.0.13 in June, hold back versions published fewer than N days ago, allowing time for malicious versions to be detected. Cooldowns are opt-in, `--cooldown 0` gets you an urgent patch immediately, and 4.1.0.beta1 brings them to `gem install`, `update`, and `outdated` too. Dependency cooldowns depend on RubyGems.org’s compact index now including when each version was published. Jenny Shen and the team at Shopify added that data through a dual-write migration and a flagged cutover, with no disruption for clients.

Read more: [Cool down before you install]({% post_url 2026-06-03-cooldown-let-new-gems-be-vetted %})

### RubyGems guides for humans and AI

The guides were rewritten in August around what you are trying to do rather than which tool does it, with a new Concepts section and security guidance covering MFA and Trusted Publishing. They are also readable by your coding assistants. Any guide URL with `.md` appended returns clean Markdown, and `llms.txt` gives an agent the whole index, so the answers you get about RubyGems can come from the current documentation.

Read more: [Making RubyGems Guides friendly to humans and AI]({% post_url 2026-08-06-making-rubygems-guides-ai-friendly %})

### Content-addressable gems

Jenny Shen, Harriet Oughton, and Gira Chawda have been working to better support native extensions in RubyGems. The work allows native extensions to be packaged per Ruby ABI, which reduces native gem size and improves maintainability for gem owners. Support for installing content-addressable gems landed in 4.1.0.beta1, and beta2 adds `gem build --content-addressable` and support in `Gem::PackageTask`. It also opens the door to building gems on RubyGems.org hardware, so native extensions would no longer have to be compiled during `bundle install`.

## RubyGems.org news

### Redesign complete

RubyGems.org has a new look. The redesign started with the About and Data pages, because the team chose stability over a big reveal. In July, it reached the rest of the site: search, gem pages, stats, profiles, API keys, and MFA settings all moved to the new design, and the blog and guides were restyled to match. The new design supports light and dark themes and works across screen sizes. Many thanks to Brian Tenggren, who got this over the line during his in-kind rotation from thoughtbot, and to Ian Taylor for the design.

### Organizations private beta

Organizations [entered private beta]({% post_url 2026-02-16-organizations-beta %}) in February. Interest from companies and teams has been strong, and a growing number are now actively using Organizations, which has given us a diverse set of perspectives to learn from. As part of the beta process, Marty met with beta users to understand the pain points and the value Organizations can bring to the publishing process. The feedback has been consistent. Participants want a simpler way to manage access across all of their company’s gems, and they value protecting their organization’s identity and verifying their affiliation.

We’ve acted on that by making onboarding easier and improving how you manage members and their access to gems. Looking ahead, we’re building more advanced publisher features that strengthen supply chain security and streamline the publishing process.

Organizations also matter because we’re exploring how they can help sustain RubyGems.org over the long term. The beta will continue into 2027 before we finalize a public rollout. If your organization would like to take part, you can [apply here](https://docs.google.com/forms/d/e/1FAIpQLScDimnhTz7u0o6kpFJEmvu6Dn-IJz5kXaW1vu5sV5qxVzoAkQ/viewform?usp=header).

### Public roadmap

RubyGems.org has a [public roadmap](https://github.com/orgs/rubygems/projects/21). It is where planned work is listed and discussed, and it is open to you. We published it in April because too much of our work had been happening quietly. If you wanted to know where things were headed, you had to catch the right conference talk or the right newsletter. The roadmap covers work at different stages, from Organizations moving toward general availability to longer-term work on security tooling, gem archival, and acceptable use policies. If something on the roadmap affects your project, tell us how. If something important is missing, raise it. The [roadmap introduction]({% post_url 2026-04-15-rubygems-org-has-a-public-roadmap %}) explains how to get involved.

## We want to hear from you

Future issues will cover releases, fixes, work in progress, and the people doing the work. Tell us what would make those updates useful, what questions about RubyGems.org we should be answering, and whose work you’d like to hear more about.

We also want to help people find a useful place to contribute. A maintainer might need help reproducing a bug or improving documentation, while someone with time to offer might not know where to start. Before we decide how Ruby Central can help, we want to understand what already works and where it falls short. If you maintain a project, what help would make a real difference? If you’ve contributed, or wanted to, what got in your way?

The best way to reach us is to email Ruby Central’s Community Manager, Kirk Haines, at [kirk@rubycentral.org](mailto:kirk@rubycentral.org). If we don’t know the answer, we’ll help you find someone who does. For questions or ideas others might want to weigh in on, start a conversation on the [Ruby Central Community Slack](https://join.slack.com/t/rubycentralcommunity/shared_invite/zt-2uh1lvotg-227RO9FJYT_C9RLNiA8Bng). Our [community page](https://rubycentral.org/community/) has more on how to get involved.

Thanks for reading, and for the work you put into Ruby.
