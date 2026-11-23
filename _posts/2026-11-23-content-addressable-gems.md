---
title: "Content-Addressable Gems: the new way of packaging precompiled binaries"
layout: post
author: Jenny Shen
author_email: jenny@rubygems.org
---

RubyGems 4.1 introduces **content-addressable gems**: precompiled gems built for a single Ruby ABI, which makes them smaller and lets maintainers support new Rubies without a new release.

### Where we are today

A gem with a native extension that ships precompiled, such as `nokogiri-1.18.9-x86_64-linux.gem`, skips compilation entirely, which speeds up `bundle install`.

But precompiled gems are larger than they need to be. A compiled extension only loads on the Ruby ABI it was built against, meaning the major and minor version of Ruby, such as `3.4`. Since a `.gem` file is identified only by its name, version, and platform, one file per platform has to bundle a copy of the extension for every supported Ruby. We'll call this a multi-ABI platform gem:

```
nokogiri-1.18.9-x86_64-linux.gem
└── lib/nokogiri/
    ├── 3.1/nokogiri.so
    ├── 3.2/nokogiri.so
    ├── 3.3/nokogiri.so
    └── 3.4/nokogiri.so
```

Most people run one Ruby, so they download four binaries to use one. Maintainers ship code to pick the right binary at load time. And when a new Ruby comes out, there is no way to add a binary for it to an existing release.

### Content-addressable gems

Content-addressable gems publish one precompiled gem per Ruby ABI. Since those files would all share the same `name-version-platform` name, each is named after a prefix of its SHA-256 checksum instead. For example, [rubydex](https://rubygems.org/gems/rubydex) 0.4.4 publishes both kinds:

```
rubydex-0.4.4-x86_64-linux.gem   # multi-ABI platform gem, every Ruby ABI
rubydex-0.4.4-aff68219.gem       # content-addressable gem, one Ruby ABI
```

The platform and Ruby ABI live in the gem's metadata instead. Each gem is smaller, and when a new Ruby is released, a maintainer can publish a build for it without cutting a new version.

Content-addressable gems are supported in RubyGems and Bundler 4.1 and later, and on RubyGems.org. Older clients skip them and keep installing multi-ABI platform gems.

This work would not have happened without Aaron Patterson, who proposed the idea, Harriet Oughton and Gira Chawda, who contributed to this feature, and friends on the RubyGems and RubyGems.org teams, who guided it through to release.

### Publishing content-addressable gems

Building content-addressable gems requires [RubyGems 4.1]({% post_url 2026-11-16-4.1.0-released %}) or later:

```console
$ gem update --system
```

If you precompile with [rake-compiler](https://github.com/rake-compiler/rake-compiler), update to 1.3.2 or later and set `content_addressable` on your extension task:

```ruby
Rake::ExtensionTask.new "my_gem", spec do |ext|
  ext.cross_compile = true
  ext.cross_platform = "x86_64-linux"
  ext.content_addressable = true
end
```

`rake cross native gem` then produces a content-addressable gem for each Ruby in `RUBY_CC_VERSION`, alongside the usual multi-ABI platform gem and the `ruby` gem:

```console
$ rake cross native gem RUBY_CC_VERSION=3.3.11:3.4.9
$ ls pkg
my_gem-1.0.gem
my_gem-1.0-x86_64-linux.gem
my_gem-1.0-5d0c19a7.gem
my_gem-1.0-e3f86b21.gem
```

Push all of them: older clients keep installing the multi-ABI platform gem, and RubyGems and Bundler 4.1 install the content-addressable one.

Without rake-compiler, use `gem build --content-addressable`. The gemspec needs a platform and a `required_ruby_version` of `~> X.Y.0`, which `--ruby-abi` can set for you:

```console
$ gem build my_gem.gemspec --platform x86_64-linux --ruby-abi 3.4 --content-addressable
```

If your gem doesn't ship precompiled yet, now is a good time to start. [cibuildgem](https://github.com/Shopify/cibuildgem) generates a GitHub Actions workflow that compiles your gem on each target platform, using rake-compiler under the hood, and pushes the results to RubyGems.org. Set `include-single-abi: true` on its action to build content-addressable gems as well.

The [Publishing Ruby ABI-specific builds](https://guides.rubygems.org/gems-with-extensions/#publishing-ruby-abi-specific-builds) section of the guides has the full details.

### Installing content-addressable gems

Once you are on RubyGems and Bundler 4.1, there is nothing to do. `gem install` and `bundle install` prefer a content-addressable gem that matches your platform and Ruby ABI, then fall back to a multi-ABI platform gem, then to the `ruby` gem. Bundler uses the version under `BUNDLED WITH` in your `Gemfile.lock`, so switch an app to 4.1 with:

```console
$ bundle update --bundler
```

Lockfiles with content-addressable gems gain two sections. The gem stays listed under its platform in `GEM`, and `CONTENT ADDRESSES` records the file and checksum to install for each Ruby ABI:

```
CONTENT ADDRESSES
  rubydex (0.4.4-x86_64-linux) 2e4d66c7 abi=3.3 sha256=2e4d66c76a4b55673e93af2d336aa7b0dcd2cb35d124a3307f2a0c7f70cc7dc1
  rubydex (0.4.4-x86_64-linux) aff68219 abi=3.4 sha256=aff682198ca8f6eabce3413c45da7c6429ed78b1414bb426a5c074cd6092c904
```

Bundler installs the row that matches the running Ruby. `RUBY ABIS` lists the Ruby ABIs the lockfile covers, much like `PLATFORMS` does for platforms:

```
RUBY ABIS
  3.3
  3.4
```

Running `bundle install` under a new Ruby adds its ABI. With a frozen lockfile, Bundler falls back to the multi-ABI platform gem if it supports that Ruby, or to the `ruby` gem if `ruby` is in `PLATFORMS`. Otherwise the install fails. To add or remove an ABI by hand:

```console
$ bundle lock --add-ruby-abi 4.0
$ bundle lock --remove-ruby-abi 3.3
```

### Looking ahead

If you maintain a gem with native extensions, try publishing content-addressable gems alongside your next release. Users on RubyGems and Bundler 4.1 and later install the content-addressable gem right away.

As more users move to 4.1 and later, gems will be able to rely on content-addressable gems alone and stop publishing a multi-ABI platform gem. We hope they become the default way to ship precompiled gems.
