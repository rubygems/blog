---
title: What's New in RubyGems/Bundler 4.1
layout: post
author: Hiroshi SHIBATA
author_email: hsbt@ruby-lang.org
---

[RubyGems and Bundler 4.1.0]({% post_url 2026-11-16-4.1.0-released %}) are out. I covered the release in a three-part series on dev.to, and this post picks two changes from each part that you are most likely to use.

## override and prune

`override` in the Gemfile rewrites version constraints that your dependencies declare, transitive ones included. When a gem caps `required_ruby_version` below the Ruby you just installed, `override "foo", required_ruby_version: :ignore_upper` gets you going without a fork. It can also drop or replace a dependency with `with:` and `in:`.

`prune` deletes Bundler's download cache and the `.git` directories of git gems after install. `bundle config set --local prune cache:git` replaces the `rm -rf` line in your Dockerfile.

Details are in [Part 1](https://dev.to/hsbt/whats-new-in-rubygemsbundler-41-part-1-override-and-prune-2ngn).

## Defenses against supply chain attacks

The credential store keeps the API key for `gem push` and Bundler's per-host credentials in the OS keychain instead of plain text files. Turn it on with `:credential_store: true` in your gemrc or `bundle config set credential_store true`.

Cooldown now works in the `gem` command too. `gem install --cooldown 7` skips versions published in the last seven days, and the `gem` command reads Bundler's cooldown setting as well.

Details are in [Part 2](https://dev.to/hsbt/whats-new-in-rubygemsbundler-41-part-2-defenses-against-supply-chain-attacks-e70).

## One codebase, fewer round trips

`gem install` no longer fetches a gemspec for every gem it installs, so `gem install activesupport` makes 25 requests instead of 40. The compact index is cached with ETags, so a repeated resolve downloads nothing.

RubyGems and Bundler now share one HTTP stack and one compact index client instead of keeping two copies, so a fix lands once.

Details are in [Part 3](https://dev.to/hsbt/whats-new-in-rubygemsbundler-41-part-3-unifying-rubygems-and-bundler-and-what-gem-install-no-300h).

Beyond these six, 4.1.0 brings many other changes that make RubyGems and Bundler nicer to use today, such as support for content addressable gems. It also has a few breaking changes. For example, `bundle clean` now exits with an error when it cannot delete a file, where it used to report success. The full changelog, breaking changes included, is in the [4.1.0 release post]({% post_url 2026-11-16-4.1.0-released %}).

To try it, run:

```shell
gem update --system
bundle update --bundler
```

The second command records Bundler 4.1.0 in your lockfile. Without it, Bundler switches back to the version your lockfile was generated with, and `override` and `prune` are not available.

Feedback is welcome, from things you notice to ideas for improving the new features. Please send it to [ruby/rubygems](https://github.com/ruby/rubygems).
