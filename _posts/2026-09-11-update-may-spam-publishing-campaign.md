---
title: "An update on the May spam-publishing campaign on rubygems.org"
layout: post
author: Colby Swandale
author_email: colby@rubygems.org
---

Following [reporting by The Wall Street Journal](https://www.wsj.com/tech/ai/cyberattack-by-rogue-ai-swarm-stokes-fears-of-out-of-control-agents-473a0352) and the publication of [research by Nightingale Collective](https://www.rubyhack.ai/), we want to clarify what we know about the spam-publishing campaign on rubygems.org in May 2026.

We take the security of rubygems.org and the community that depends on it seriously. Our team reviewed the activity and discussed the findings with researchers from Nightingale Collective.

The campaign involved newly registered accounts publishing spam packages. Socket previously documented related activity under the name [GemStuffer](https://socket.dev/blog/gemstuffer). As described in our [May status updates](https://status.rubygems.org/incidents/cytf062tkwtt), we temporarily paused new account registrations, blocked and removed the accounts responsible, and yanked more than 500 malicious packages. Gem installs and pushes for existing users remained unaffected, and registrations reopened on May 16.

Nightingale Collective’s research describes packages designed to use shared Ruby infrastructure to run code, retrieve publicly available web data, and publish that data back to rubygems.org. The researchers also identified code intended to obtain other users’ API keys. Our investigation found no evidence that these attempts succeeded.

The researchers attribute the activity to OpenAI agents. Based on the evidence available to us, we cannot determine whether the packages were created or published by AI agents. Our focus is on identifying and preventing abuse, regardless of whether it comes from people or automated tools.

We appreciate Nightingale Collective’s engagement with our team. Responding to abuse requires time and resources from the people maintaining package repositories, alongside their everyday work of keeping these services secure and reliable for the community.

Colby Swandale  
Technical Lead, Ruby Central  
On behalf of the rubygems.org team
