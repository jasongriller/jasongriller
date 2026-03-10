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
