---
title: "Making RubyGems Guides friendly to humans and AI"
layout: post
author: Hiroshi SHIBATA
author_email: hsbt@ruby-lang.org
---

The RubyGems guides at [guides.rubygems.org](https://guides.rubygems.org) are read by more than humans these days. AI agents fetch them to answer questions about building, publishing, and installing gems. Evil Martians' [Ruby/Rails LLM discoverability scorecard](https://ruby.evilmartians.com/) asks how easily an agent can find and read Ruby documentation, and the guides had no good answer. There was no machine readable index, and no way to get a page without its navigation and markup. This week we merged three changes that close both gaps.

The first change adds `sitemap.xml` and `robots.txt` ([rubygems/guides#523](https://github.com/rubygems/guides/pull/523)), giving crawlers a complete map of the site.

The second adds [`llms.txt`](https://guides.rubygems.org/llms.txt) and [`llms-full.txt`](https://guides.rubygems.org/llms-full.txt) ([rubygems/guides#525](https://github.com/rubygems/guides/pull/525)). `llms.txt` lists every guide with a one-line description in about 10 KB, so an agent can fetch it once and jump straight to the page it needs. `llms-full.txt` concatenates the full text of all guides into a single 448 KB document for tools that want everything at once.

The third serves raw Markdown for every page ([rubygems/guides#524](https://github.com/rubygems/guides/pull/524)). Append `.md` to any guide URL, such as [what-is-a-gem.md](https://guides.rubygems.org/what-is-a-gem.md), and you get the rendered Markdown source with no navigation or markup. That cuts page size to between a half and an eighth of the HTML. `what-is-a-gem` is 26.7 KB as HTML and 3.3 KB as Markdown. Each HTML page also links its Markdown twin via `rel="alternate" type="text/markdown"`.

To try it, hand your agent the URL `https://guides.rubygems.org/llms.txt` and ask a RubyGems question. One fetch gives it a table of contents, and every follow-up fetch spends tokens on content instead of markup.

### New formats need fresh content

A machine readable feed is only as useful as the writing behind it. Serving an agent guidance that still mentioned freenode IRC and Ruby 1.8 path layouts would defeat the purpose, so the content was overhauled alongside the formats. The guides are reorganized around tasks instead of tools, into four sections named Getting Started, Guides, Concepts, and Reference, with full-text search powered by Pagefind. The new Concepts section explains how the system works, from [dependency resolution](https://guides.rubygems.org/dependency-resolution/) to platforms and native gems. Reference now documents [`.gemrc` configuration](https://guides.rubygems.org/configuration/) and environment variables for the first time, and absorbs the Bundler man pages. The [security guide](https://guides.rubygems.org/security/) was rewritten around MFA, Trusted Publishing, lockfile checksums, and [cooldown]({% post_url 2026-06-03-cooldown-let-new-gems-be-vetted %}).

None of this is a one-off. Ruby Central set out Project DREAM (Driving Ruby's Evolution to AI Maturity) in [A New Chapter for Ruby Central](https://rubycentral.org/news/a-new-chapter-for-ruby-central/), and documentation is where that work is easiest to see. Both audiences want the same things from a guide, which are clear structure, current facts, and a page you can reach without wading through anything else. There is more to do, and we will keep at it.
