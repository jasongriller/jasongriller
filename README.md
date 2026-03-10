### Hi there 👋

<!--
**jasongriller/jasongriller** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->

Here's a full breakdown of everything changed and why:



1. lambdas/sprint-notes/src/index.js

Single-attempt mode for large CSVs
When a structured CSV has >10 items, maxGenerationAttempts is set to 1 instead of the default (3). Previously, if validation failed after the first Bedrock call, it would retry — doubling or tripling the wall-clock time and almost guaranteeing a 504. Now it always returns something on the first attempt.

First-draft fallback
If running in single-attempt mode and validation finds issues, instead of throwing an error it returns the result anyway with the issues surfaced as inputWarnings. The user gets output instead of a blank failure.

Deep review threshold tightened
shouldRunDeepReview — the deep review is an extra Bedrock call after the initial generation. For structured mode, it used to trigger when the prompt was under 14,000 chars. That was lowered to 10,000 chars, so fewer large requests burn the extra Bedrock call time.

KB retrieval threshold tightened
shouldRetrieveSemanticContext — retrieving knowledge base context is another async Bedrock call. For structured mode it used to always run for ≤15 items. Lowered to ≤10 items, skipping it for medium-large uploads.

Observability
Added isLargeStructured and maxGenerationAttempts to both the CloudWatch log block and the API response inputSummary, so you can see in logs whether single-attempt mode fired.



2. terraform/modules/api-gateway/main.tf

Added timeout_milliseconds = 29000 (explicitly, with a TODO comment) to aws_api_gateway_integration.sprint_notes_generate_post.

This is currently set to 29000 because the AWS quota increase is still pending. Once support case 177308460706240 is approved, this gets bumped to 60000 and Terraform re-applied. That's the real fix — API Gateway will wait 60s instead of cutting the connection at 29s.



3. terraform/modules/sprint-notes/main.tf

Bumped the Lambda timeout from 60s → 90s. Lambda always needs to be set higher than the API Gateway integration timeout, otherwise Lambda itself could time out before API Gateway does. With API Gateway going to 60s, Lambda needed headroom above that.



Root cause summary

The 504 was caused by API Gateway's hard 29-second integration timeout — a limit that exists independently of Lambda's own timeout. Lambda could be perfectly healthy and still working, but API Gateway would terminate the HTTP connection at 29s and return 504 to the browser. The Lambda code changes reduce how long generation takes; the quota increase removes the hard ceiling entirely.


# AI Team — Problem Intake Document

> **Purpose:** Fill out this document when requesting the AI team to solve a problem. The more context you provide, the faster and more accurately we can deliver a solution.

---

## Requester Info

- **Name:**
- **Team / Department:**
- **Date Submitted:**
- **Priority:** [ ] P1 — Critical  [ ] P2 — High  [ ] P3 — Medium  [ ] P4 — Low
- **Target Completion Date:**

---

## The 3 Pillars

### Pillar 1 — Why'd It Change? (Context & History)

_Help us understand what led to this request. What happened that made you reach out?_

- **What changed recently?** _(e.g. new regulation, process broke, business shift, user complaint, leadership ask)_


- **When did the change happen or when was the problem first noticed?**


- **What was the previous state?** _(How did things work before? What was the old process/tool/workflow?)_


- **Why did it change?** _(Root cause if known — was it intentional or a side effect?)_


- **Who or what is affected?** _(Teams, users, customers, downstream systems)_


---

### Pillar 2 — What's New in the System? (Current State)

_Give us a snapshot of the world as it exists today._

- **What systems/tools are involved?** _(Applications, databases, APIs, platforms — list everything relevant)_


- **What does the current workflow look like?** _(Step-by-step, even if it's manual or broken)_


- **What data is available?** _(Data sources, formats, access — include links/paths if possible)_


- **Any recent system updates or deployments?** _(Patches, migrations, new integrations, config changes)_


- **Known constraints or limitations?** _(Security, compliance, budget, tech stack restrictions, timelines)_


---

### Pillar 3 — What Do You Need Us to Solve? (The Ask)

_Be as specific as possible about what success looks like._

- **Problem statement (1-3 sentences):** _(What is the core problem?)_


- **What is your ideal outcome?** _(If we nailed it, what would the result look like?)_


- **How do you measure success?** _(KPIs, metrics, acceptance criteria — how will we know it worked?)_


- **Have you tried anything already?** _(Previous attempts, workarounds, things that didn't work)_


- **What happens if we don't solve this?** _(Impact of inaction — helps us prioritize)_


---

## Supporting Materials

_Attach or link anything that helps us understand the problem faster._

- [ ] Screenshots / screen recordings
- [ ] Sample data or example files
- [ ] Architecture diagrams or flowcharts
- [ ] Error logs or stack traces
- [ ] Existing documentation or SOPs
- [ ] Links to relevant tickets, threads, or prior work

---

## Stakeholders & Communication

- **Primary point of contact:**
- **Other stakeholders who should be looped in:**
- **Preferred communication channel:** _(Teams, email, Slack, standup, etc.)_
- **How often do you want progress updates?** _(Daily, weekly, at milestones)_

---

## AI Team Use Only _(Do not fill out)_

- **Assigned To:**
- **Feasibility Assessment:**
- **Estimated Effort:**
- **Approach / Solution Type:**
- **Status:** [ ] Intake Review  [ ] In Discovery  [ ] In Progress  [ ] Delivered  [ ] Closed

---

> **Tip:** A well-filled intake saves everyone time. If you're unsure about a field, write what you *do* know and flag what you're uncertain about — that's still valuable context.

