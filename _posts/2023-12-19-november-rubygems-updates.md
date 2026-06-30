---
title: November 2023 RubyGems Updates
layout: post
author: Gift Egwuenu
author_email: laurandidi21@gmail.com
---

Welcome to the RubyGems monthly update! As part of our efforts at Ruby Central, we publish a recap of the work that we’ve done the previous month. Read on to find out what updates were made to RubyGems and RubyGems.org in November.

## RubyGems News

This month in RubyGems, we released RubyGems [3.4.22](https://github.com/rubygems/rubygems/blob/master/CHANGELOG.md#3422--2023-11-09) and Bundler [2.4.22](https://github.com/rubygems/rubygems/blob/master/bundler/CHANGELOG.md#2422-november-9-2023). These updates mark significant milestones in our continuous effort to enhance the Ruby development experience.

One of the key changes is a boost in performance, achieved by eliminating the need for regular expression matches in `Gem::Platform.local` ([#7104](https://github.com/rubygems/rubygems/pull/7104)). Additionally, we shipped an update of the SPDX license list, ensuring that RubyGems is in line with the most recent industry standards ([#7040](https://github.com/rubygems/rubygems/pull/7040)).

Another important update: The YAML serializer has been enhanced to correctly handle empty arrays ([#7099](https://github.com/rubygems/rubygems/pull/7099)). The search process has also been refined, now effectively ignoring `.gem` files that are not in tar format ([#7095](https://github.com/rubygems/rubygems/pull/7095)). The update also brings a new feature that allows users to uninstall multiple versions of the same gem at once, making gem management more efficient ([#7063](https://github.com/rubygems/rubygems/pull/7063)).

Some other improvements that landed into our repo this month that are NOT included in the above releases are:

- an added response body on `fetch_http error` ([#7148](https://github.com/rubygems/rubygems/pull/7148)).
- a more robust writability check for gem home ([#7211](https://github.com/rubygems/rubygems/pull/7211)).
- an update to the Magnus library in the Rust extension gem template ([#7204](https://github.com/rubygems/rubygems/pull/7204)).
- an update to gem uninstall error reporting ([#7149](https://github.com/rubygems/rubygems/pull/7149)).
- an added `--json` output option to bundle-outdated ([#7167](https://github.com/rubygems/rubygems/pull/7167)).
- a fix for invalid platform removal omitting adjacent platforms ([#7170 ](https://github.com/rubygems/rubygems/pull/7170)).
- fixed universal lockfiles regression  ([#7177](https://github.com/rubygems/rubygems/pull/7177)).
- dropped Ruby 2.6 and 2.7 support ([#7116](https://github.com/rubygems/rubygems/pull/7116)).
- fixed bundle install `--system` deprecation advice ([#7190](https://github.com/rubygems/rubygems/pull/7190)).
- no longer remembering cli flags like` --jobs` or ` --retry` in configuration ([#7191](https://github.com/rubygems/rubygems/pull/7191)).
- an added option for missing `--prefer-local` to Synopsis in bundle-install.1.ronn ([#7194](https://github.com/rubygems/rubygems/pull/7194)).
- allowing auto-install to install missing git gems ([#7197](https://github.com/rubygems/rubygems/pull/7197)).
- ensuring explicit requirement of `rubygems` ([#7139](https://github.com/rubygems/rubygems/pull/7139)).


In November, RubyGems gained [113 new commits](https://github.com/rubygems/rubygems/compare/master@%7B2023-11-01%7D...master@%7B2023-11-31%7D) contributed by 23 authors. There were 1,875 additions and 56,824 deletions across 1,496 files.

## RubyGems.org News

The updates to RubyGems.org this month reflect a strong commitment to improving user experience, enhancing security, and modernizing the platform. 

One of the updates made was to upgrade to Postgresql 12 and updated production and staging `DATABASE_URL` to PostgreSQL 12 instance ([#4245](https://github.com/rubygems/rubygems.org/pull/4245), [#4256](https://github.com/rubygems/rubygems.org/pull/4256)).

Some other improvements that landed into our repo this month that are NOT included in the above releases are:

- a fix for deletion `version_id` backfill when a user is deleted ([#4259](https://github.com/rubygems/rubygems.org/pull/4259)).
- an added `version_id` column to Deletions ([#4254](https://github.com/rubygems/rubygems.org/pull/4254)).
- an added [trusted publishers](https://blog.rubygems.org/2023/12/14/trusted-publishing.html) feature to help automate gem publishing ([#4239](https://github.com/rubygems/rubygems.org/pull/4239)).
- fixed `api_key_created email` when API key belongs to an OIDC id token ([#4233](https://github.com/rubygems/rubygems.org/pull/4233)).
- an added `maintenance_task` to backfill info files into S3 ([#4232](https://github.com/rubygems/rubygems.org/pull/4232)).
- use of an uncached query to compute compact index info in jobs ([#4231](https://github.com/rubygems/rubygems.org/pull/4231)).
- an added job to refresh all OIDC provider configs every 30m ([#4211](https://github.com/rubygems/rubygems.org/pull/4211))
- extraction of verified session logic into a concern ([#4210](https://github.com/rubygems/rubygems.org/pull/4210)).
- updated installation instructions OS X ([#4203](https://github.com/rubygems/rubygems.org/pull/4203)).
- an upgrade of shoryuken to version 5.x ([#4166](https://github.com/rubygems/rubygems.org/pull/4166)).

In November, RubyGems.org gained [83 new commits](https://github.com/rubygems/rubygems.org/compare/master@%7B2023-11-01%7D...master@%7B2023-11-31%7D) contributed by 7 authors. There were 950 additions and 501 deletions across 109 files.

## Thank you

Thank you to all the contributors of RubyGems and RubyGems.org for this month! Your contributions are greatly appreciated, and we are grateful for your support.

### Contributors to RubyGems:

- [@segiddins](https://github.com/segiddins) Samuel Giddins
- [@simi](https://github.com/simi) Josef Šimánek
- [@hanyang-tony](https://github.com/hanyang-tony) Hanyang tony
- [@colby-swandale](https://github.com/colby-swandale) Colby Swandale
- [@deivid-rodriguez](https://github.com/deivid-rodriguez) David Rodríguez
- [@voxik](https://github.com/voxik) Vít Ondruch
- [@Paul-Bob](https://github.com/Paul-Bob) Paul Bob
- [@nobu](https://github.com/nobu) Nobuyoshi Nakada
- [@eregon](https://github.com/eregon) Benoit Daloze
- [@hsbt](https://github.com/hsbt) Hiroshi Shibata
- [@duckinator](https://github.com/duckinator) Ellen Marie Dash
- [@martinemde](https://github.com/martinemde) Martin Emde
- [@Bo98](https://github.com/Bo98) Bo Anderson
- [@nevinera](https://github.com/nevinera) Eric Mueller
- [@Maumagnaguagno](https://github.com/Maumagnaguagno) Mau Magnaguagno
- [@olleolleolle](https://github.com/olleolleolle) Olle Jonsson
- [@ggmichaelgo](https://github.com/ggmichaelgo) Michael Go
- [@adrianthedev](https://github.com/adrianthedev) Adrian Marin
- [@kstevens](https://github.com/kstevens715) Kyle Stevens
- [@dearblue](https://github.com/dearblue) Dearblue


### Contributors to RubyGems.org:

- [@segiddins](https://github.com/segiddins) Samuel Giddins
- [@simi](https://github.com/simi) Josef Šimánek
- [@colby-swandale](https://github.com/colby-swandale) Colby Swandale
- [@martinemde](https://github.com/martinemde) Martin Emde
- [@hsbt](https://github.com/hsbt) Hiroshi Shibata
- [@jp524](https://github.com/jp524) Jade

---
Learn more about contributing to RubyGems by visiting the RubyGems Contributing Guide. We welcome all kinds of contributions, including bug fixes, feature implementation, writing and updating documentation, and bug triage.
