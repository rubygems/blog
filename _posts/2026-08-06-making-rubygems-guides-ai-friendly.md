---
title: "Making RubyGems Guides friendly to humans and AI"
layout: post
author: Hiroshi SHIBATA
author_email: hsbt@ruby-lang.org
---

The RubyGems guides at [guides.rubygems.org](https://guides.rubygems.org) are read by more than humans these days. AI agents fetch them to answer questions about building, publishing, and installing gems. This week we merged three changes that make those fetches far more efficient, as one step in an ongoing effort to make the guides easier to use for everyone.

The first change adds `sitemap.xml` and `robots.txt` ([rubygems/guides#523](https://github.com/rubygems/guides/pull/523)), giving crawlers a complete map of the site.

The second adds [`llms.txt`](https://guides.rubygems.org/llms.txt) and [`llms-full.txt`](https://guides.rubygems.org/llms-full.txt) ([rubygems/guides#525](https://github.com/rubygems/guides/pull/525)). `llms.txt` lists every guide with a one-line description in about 10 KB, so an agent can fetch it once and jump straight to the page it needs. `llms-full.txt` concatenates the full text of all guides into a single 448 KB document for tools that want everything at once.

The third serves raw Markdown for every page ([rubygems/guides#524](https://github.com/rubygems/guides/pull/524)). Append `.md` to any guide URL, such as [what-is-a-gem.md](https://guides.rubygems.org/what-is-a-gem.md), and you get the rendered Markdown source with no navigation or markup. That cuts page size to between a half and an eighth of the HTML. `what-is-a-gem` is 26.7 KB as HTML and 3.3 KB as Markdown. Each HTML page also links its Markdown twin via `rel="alternate" type="text/markdown"`.

To try it, hand your agent the URL `https://guides.rubygems.org/llms.txt` and ask a RubyGems question. One fetch gives it a table of contents, and every follow-up fetch spends tokens on content instead of markup.

Evil Martians' [Ruby/Rails LLM discoverability scorecard](https://ruby.evilmartians.com/) prompted this work, and it follows the direction of Project DREAM (Driving Ruby's Evolution to AI Maturity) that Ruby Central described in [A New Chapter for Ruby Central](https://rubycentral.org/news/a-new-chapter-for-ruby-central/). Improvements for human readers are underway in parallel, including a reorganized top page and guide structure and full-text search powered by Pagefind. More to come.
