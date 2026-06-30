---
title: What's New in RubyGems/Bundler 4
layout: post
author: Hiroshi SHIBATA
author_email: hsbt@ruby-lang.org
---

[Ruby 4.0.0 was released on December 25, 2025](https://www.ruby-lang.org/en/news/2025/12/25/ruby-4-0-0-released/), and RubyGems/Bundler 4.0.3 is now bundled with Ruby 4.0.0.

Since [my previous post focused on migration and compatibility concerns](https://blog.rubygems.org/2025/12/03/upgrade-to-rubygems-bundler-4.html), I'd like to highlight some of the exciting new features in this release.

## Parallelization of C-extension Gem Builds

[Add `MAKEFLAGS=-j` by default before compiling](https://github.com/ruby/rubygems/pull/9131)

When installing gems with C extensions (such as `mysql2` or `pg`), RubyGems now automatically adds `MAKEFLAGS=-j` to the make command for parallel execution. Users previously had to manually configure this themselves. By leveraging multi-core CPUs by default, installation times are significantly reduced.

By default, RubyGems uses `Etc.nprocessors + 1`, fully utilizing available CPU cores during compilation. You can override this by setting `MAKEFLAGS` explicitly (e.g., `MAKEFLAGS=-j2`).

While convenient for local development, we discovered after implementation that in containerized environments like CircleCI, `Etc.nprocessors` returns the host server's CPU count rather than the container's allocation. This caused failures when jobs were assigned `-j32` despite having only 2 available CPUs. To address this, we've introduced the following improvements.

## Unified Parallel Job Execution Options

[Pass down value of `BUNDLE_JOBS` to RubyGems before compiling & introduce a new `gem install -j` flag](https://github.com/ruby/rubygems/pull/9171)

To prevent resource exhaustion in virtual environments, a new `-j` flag for `gem install` and automatic passing of `BUNDLE_JOBS` to RubyGems have been introduced.

Now when you set `BUNDLE_JOBS=4`, `bundle install` runs with 4-way parallelism, and C extensions are built with `-j4`. In resource-constrained CI environments, specifying 1 or 2 prevents resource exhaustion.

However, even with these changes, worst-case scenarios remain (where `BUNDLE_JOBS=4` could consume up to 16 CPU cores). To address this, we're exploring a [GNU Make jobserver](https://www.gnu.org/software/make/manual/html_node/POSIX-Jobserver.html)—a mechanism where a server controls available CPUs and jobs query it before running.

While the specifics are still in development, this should eliminate worst-case scenarios. We've also learned that Go's `GOMAXPROCS` handles containerized environments like CircleCI by reading CPU info from cgroup:

[Feature #21797: Improve Etc.nprocessors for cgroup environment](https://bugs.ruby-lang.org/issues/21797)

By incorporating this approach into Ruby core and combining it with a jobserver, we expect to achieve optimal build speeds while fully utilizing available CPUs. Stay tuned!

## Increased Connection Pool and Efficient Network Communication

[Increase connection pool to allow for up to 70% speed increase on `bundle install`](https://github.com/ruby/rubygems/pull/9087)

The default connection pool for Bundler and RubyGems network requests has been expanded to 5 parallel connections. This improvement reportedly reduces `bundle install` times by up to 70%.

[Adjust the API_REQUEST_LIMIT to make less network roundtrip](https://github.com/ruby/rubygems/pull/9071)

Additionally, `API_REQUEST_LIMIT` (which controls batch sizes of dependency information) has increased from 50 to 100, doubling the gems per request. For example, a Gemfile with 400 dependencies now requires just 4 network requests instead of 8.

## Pattern Matching Support for Gem::NameTuple and Gem::Platform

[Add pattern matching support to Gem::Platform](https://github.com/ruby/rubygems/pull/9062)

When writing scripts that need to inspect gem names, versions, or platforms, you can now use Ruby's pattern matching for cleaner and safer logic.

As shown in the GitHub PR, you can write platform-specific logic cleanly using pattern matching on `Gem::Platform`:

```ruby
case platform
in cpu: "x86_64", os: "linux"
  install_linux_x64
in cpu: "arm64", os: "darwin"
  install_macos_arm
else
  # ...
end
```

Pattern matching support for `Gem::Version` is still under consideration and may not move forward.

[Add pattern matching support to Gem::Version](https://github.com/ruby/rubygems/pull/9060)

The proposal suggested decomposing versions into `major`, `minor`, and `build` components, but real-world versions include strings like `4.0.0.beta3` or `4.0.0.beta.3`. It would be inconsistent for these to equal `4.0.0`, and equally confusing for `beta3` and `beta.3` to be treated as equivalent. Additionally, since `build` is sometimes called `tiny` or `patch` and lacks a clear definition, I'm hesitant about introducing this feature.

## JSON Output for bundle list

[Introduce `bundle list --format=json`](https://github.com/ruby/rubygems/pull/8728)

A new `--format=json` option for `bundle list` makes it easy to integrate Bundler with external tools.

When analyzing a project's gem dependencies programmatically (via CI tools, security scanners, etc.), JSON output eliminates the need to parse text, greatly simplifying integration. While Bundler has parsers for Gemfiles and lockfiles, they're essentially Ruby code or custom formats—so being able to process them with tools like `jq` is a welcome addition.

```bash
$ bundle list --format=json | jq -r '.gems[] | select(.name == "json") | .version'
2.18.0
```

## bundle install Without Generating a Lockfile

[Add support for lockfile in Gemfile and bundle install --no-lock](https://github.com/ruby/rubygems/pull/9059)

The `bundle install --no-lock` flag now skips lockfile generation, which is convenient when you want to test dependency resolution without creating or modifying lockfile artifacts.

## Specifying the Generated Lockfile Name

[Support bundle install --lockfile option](https://github.com/ruby/rubygems/pull/9111)

You can now specify a custom lockfile name using `bundle install --gemfile=foo --lockfile=bar`. This defaults to the `.lock` extension if not specified, but is useful when working with variants like `Gemfile.next` or `Gemfile.rails81`.

## Creating Go-extension Gems

[Add `--ext=go` to `bundle gem`](https://github.com/ruby/rubygems/pull/8183)

RubyGems' gem scaffolding and build tools previously supported C and Rust extensions. Support for Go has now been added. You can create a Go-extension gem template with `bundle gem --ext=go foo`.

## Using did_you_mean for Bundler Typo Checks

[Use DidYouMean::SpellChecker for gem suggestions in Bundler](https://github.com/ruby/rubygems/pull/3857)

Bundler's typo detection and command suggestions previously used a custom implementation. This has now been updated to use `did_you_mean`, keeping it consistent with other Ruby tools.

## Summary

This post highlighted some major new features in RubyGems/Bundler 4.0. The project consists of two main components: the "resolver" (which handles dependency resolution) and the "CLI/UI" (which manages user interaction).

Historically, RubyGems and Bundler have maintained separate codebases for their resolvers despite performing similar tasks. Looking ahead to version 4.1, we plan to gradually consolidate and unify these components.

I’m looking forward to working with you all to make RubyGems and Bundler even better in 2026!