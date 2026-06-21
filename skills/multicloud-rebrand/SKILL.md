---
name: multicloud-rebrand
description: Use when rebranding or generalizing a product hardcoded to ONE cloud/vendor (e.g. AWS-first) into a multi-vendor one, without breaking accuracy for existing single-vendor customers or fabricating per-cloud detail you don't have.
---

# Rebrand a single-cloud product to multicloud

## Overview

An AWS-first (or any single-vendor) product accretes the vendor's name into copy, scoring, prompts, and program logic. Going multicloud is mostly **reposition + parameterize, not rebuild** — but the two traps are (a) fabricating per-cloud detail you don't actually have, and (b) erasing accurate vendor-specific value your current customers rely on.

## The discipline

1. **Separate chrome from working content.**
   - PLATFORM CHROME (product name, hero, identity, generic labels) → cloud-NEUTRAL. "forges your AWS deals" → "forges your cloud deals"; "AWS plays" → "Funding plays".
   - WORKING CONTENT (specific programs, services, plays) → driven by the PARTNER's cloud, not hardcoded. Lead with the partner's own cloud.
2. **Add a partner-cloud keystone.** One function/field = "what cloud(s) does this partner sell" (default to the incumbent for existing partners). Everything per-cloud reads from it. Adding a cloud becomes data, not a code change.
3. **Keep vendor-specific tools accurate, scoped, and labeled.** A deep AWS-only workflow (e.g. ACE / Partner Central paperwork) is CORRECT for AWS partners — keep it, label it AWS-specific, and build per-cloud siblings later. Do not gut it into vagueness.
4. **NEVER fabricate per-cloud content.** If you lack real Azure/GCP program names, funding %s, or service equivalents, make the copy cloud-RELATIVE ("on the partner's cloud") instead of inventing specifics. Fabricated equivalents are worse than honest generality; fill them with real data (or a real partner) later.
5. **Neutralize the scoring too.** A shared signal that rewards one cloud over others bakes in the bias — score every vendor equally and move the "home vs migrate" judgment into a per-partner layer.

## Red flags

| Thought | Reality |
|---------|---------|
| "Replace AWS with 'cloud' everywhere" | You'll erase accurate, valuable vendor-specific tools current customers use. Scope, don't scorch. |
| "Map every AWS service to Azure/GCP now" | If you can't ground the equivalents, you're fabricating. Go cloud-relative; fill real later. |
| "The voice should be generic" | Generic = weaker. Make it fluent across clouds and LEAD with the partner's own. |
| "It's just copy" | Cloud framing also lives in scoring, prompts, and program logic. Find all of it, not just the strings. |

## Worked example (Alloy)

Five phases: copy/labels neutral → cloud-note framing → Smith's voice (app prompts + the LLM-gateway prompt) reframed to "cloud co-worker, fluent across AWS/Azure/GCP, leads with the partner's cloud" → ICP score rebalanced (aws = azure = gcp) → `partnerClouds(project)` keystone + cloud-relative vendor map. Kept AWS-specific BY DESIGN: the funding-paperwork (ACE/PDM) and migration-assessment tools (accurate for today's AWS partners; per-cloud siblings = future grounded work).
