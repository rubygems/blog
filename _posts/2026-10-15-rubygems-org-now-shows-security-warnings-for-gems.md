---
title: "rubygems.org now shows security warnings for gems"
layout: post
author: Anton Bondaronok
author_email: anton@rubygems.org
---

Most of us have a routine for adding a gem: search for it, open its page on [rubygems.org](https://rubygems.org), glance at the download count and the number of days since the last release, then copy the Gemfile line.

The whole process takes half a minute. But until now, [rubygems.org](https://rubygems.org) did not tell you whether the version you were about to install had a known security issue.

That information has been public for years. Sources such as the GitHub Advisory Database, OSV and Ruby Advisory Database already track vulnerabilities. It just wasn’t shown on the page where many developers make the decision to add a dependency.

Now it is.

![Security advisories shown on a gem version page](/images/security-warnings-for-gems.png)

## How security advisories appear on gem pages

When you view a gem version affected by a known security issue, [rubygems.org](https://rubygems.org) displays a warning or critical alert based on its severity. Each advisory includes a severity level, a short summary and a link to the full report. If multiple advisories affect that version, the page lists each one so you can assess how the issues affect your application.

Warnings follow the version you’re viewing. If the latest release includes the fix, the gem’s main page no longer shows that warning. Of course, that doesn’t mean the gem is 100% safe. We recommend that users still exercise normal caution when installing new gems. Older releases that are still affected get a warning icon in the versions list, so you can see which versions are affected.

The first milestone of this work covers the website only, so `gem install` and `bundle install` do not change yet. The warnings help you review known security issues before running `bundle install` or `gem install`.

## Where the vulnerability data comes from

[OSV](https://osv.dev/) is the main source of advisory data for this feature. It is an open database that collects vulnerability reports from sources such as the GitHub Advisory Database and provides them in a shared format. [rubygems.org](https://rubygems.org) pulls a fresh copy of this data every hour.

OSV also includes malicious package reports from the [OpenSSF Malicious Packages](https://github.com/ossf/malicious-packages) project. [rubygems.org](https://rubygems.org) labels the affected gem versions as malware.

## For gem maintainers: what to do if your gem has a warning

[rubygems.org](https://rubygems.org) does not write or edit advisories. It displays advisory data collected by OSV from sources such as the GitHub Advisory Database. If a warning about your gem is incorrect, request a correction from the original source.

For GitHub advisories, you can use the link on the advisory page to suggest improvements. For malware reports, you can contact the OpenSSF project. Once the correction reaches OSV, [rubygems.org](https://rubygems.org) picks it up within about an hour.

After releasing a fix, check that the advisory correctly identifies the fixed version. If it still lists your new release as affected, [rubygems.org](https://rubygems.org) will continue to show a warning until the advisory is corrected.

## What’s next for gem security data

The next milestone is to add security advisory data to the Compact Index, the API that Bundler and RubyGems use to retrieve gem and version information. This will make the data available to those tools so they can warn users about affected versions.

Every warning shown on [rubygems.org](https://rubygems.org) depends on the people who maintain the advisory databases behind it. Thanks to everyone doing that work.
