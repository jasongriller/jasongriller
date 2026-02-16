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




Got all the details. Let me think through this end-to-end.

  ---
  The Architecture

  Here's what I'm thinking — a new repo-digest Lambda that runs on a daily schedule:

  EventBridge (daily cron)
    │
    ▼
  repo-digest Lambda
    │
    ├─ 1. Fetch ADO creds from Secrets Manager (same secret, already exists)
    ├─ 2. Shallow-clone AIE CODE repo (AIE/_git/AIE, not AIE.wiki)
    ├─ 3. Check: has anything changed since last run?
    │      └─ Compare HEAD commit hash vs. hash stored on the existing S3 doc's metadata
    │      └─ If same → exit early, no work needed
    ├─ 4. Read only process-relevant files:
    │      └─ docs/**, pipelines/**, release-definitions/**, data-dictionary/**
    ├─ 5. Get git log (last 30 days) for "Recent Changes" section
    ├─ 6. Feed everything to Bedrock (ConverseCommand, Nova Pro) with a prompt like:
    │      "Given these repo files, generate a structured summary of the team's
    │       processes and workflows — release process, CI/CD, BA workflow, QA workflow..."
    ├─ 7. Save generated markdown to S3:
    │      └─ knowledge-base-docs/context/aie-team-processes.md
    │      └─ metadata: { status: 'active', last-commit-hash: 'abc123' }
    ├─ 8. Trigger Bedrock ingestion (curated data source)
    └─ Done


  Why This Works

  No new tab — the generated doc lands in knowledge-base-docs/context/, which is the existing Curated tab. It'll show up alongside your other curated docs like any other document.

  No raw files — Bedrock distills the pipeline YAML, release definitions, and docs into readable prose about how the team works.

  Auto-updates — runs daily. If nothing changed in the AIE repo (same commit hash), it skips entirely. Zero wasted Bedrock calls.

  Existing permissions cover everything:

  ┌───────────────────────────────────┬──────────────────────────────────────────────────┐
  │               Need                │                 Already have it?                 │
  ├───────────────────────────────────┼──────────────────────────────────────────────────┤
  │ ADO credentials (Secrets Manager) │ Yes — same secret used by wiki-sync              │
  ├───────────────────────────────────┼──────────────────────────────────────────────────┤
  │ Git clone (isomorphic-git)        │ Yes — same pattern                               │
  ├───────────────────────────────────┼──────────────────────────────────────────────────┤
  │ Bedrock ConverseCommand           │ Yes — shared IAM role has bedrock:InvokeModel    │
  ├───────────────────────────────────┼──────────────────────────────────────────────────┤
  │ S3 write to knowledge-base-docs/  │ Yes — shared role has full bucket access         │
  ├───────────────────────────────────┼──────────────────────────────────────────────────┤
  │ Bedrock ingestion trigger         │ Yes — StartIngestionJobCommand permission exists │
  └───────────────────────────────────┴──────────────────────────────────────────────────┘

  Cost: ~$1-2/month. One Bedrock call per day (skipped if no changes), tiny Lambda runtime.

  What the Generated Doc Would Look Like

  # AIE Team Processes & Workflow
  *Auto-generated from AIE repository — Last updated: 2026-02-16 — Commit: d88cdb0*

  ## Release Process
  [Distilled from release-definitions/ — how releases are cut, environments, approvals]

  ## CI/CD Pipeline
  [Distilled from pipelines/ — build stages, deployment gates, environments]

  ## Development Workflow
  [Distilled from docs/ — branching strategy, PR process, code review]

  ## Data Dictionary
  [Distilled from data-dictionary/ — key entities, relationships, field definitions]

  ## QA Process
  [Extracted from docs and pipeline stages — test gates, automation, sign-off]

  ## BA Process
  [Extracted from docs — requirements gathering, story writing conventions]

  ## Recent Changes (Last 30 Days)
  - 2026-02-16 — @salman — Updated release pipeline for staging gate
  - 2026-02-14 — @john — Added QA checklist to docs
  - ...

  Implementation Pieces

  ┌─────────────────────┬────────────────────────────────────────────────────────────────────┐
  │        Piece        │                               Where                                │
  ├─────────────────────┼────────────────────────────────────────────────────────────────────┤
  │ Lambda code         │ lambdas/repo-digest/src/index.js (~200-300 lines)                  │
  ├─────────────────────┼────────────────────────────────────────────────────────────────────┤
  │ Terraform module    │ terraform/modules/repo-digest/main.tf                              │
  ├─────────────────────┼────────────────────────────────────────────────────────────────────┤
  │ Module wiring       │ New block in terraform/main.tf                                     │
  ├─────────────────────┼────────────────────────────────────────────────────────────────────┤
  │ EventBridge rule    │ cron(35 0 * * ? *) (daily, 30 min after analytics cron to stagger) │
  ├─────────────────────┼────────────────────────────────────────────────────────────────────┤
  │ Bedrock data source │ None needed — doc goes into existing knowledge-base-docs/ prefix   │
  ├─────────────────────┼────────────────────────────────────────────────────────────────────┤
  │ New IAM role        │ None needed — shared role covers everything                        │
  ├─────────────────────┼────────────────────────────────────────────────────────────────────┤
  │ New DynamoDB table  │ None needed — commit hash stored as S3 object metadata             │
  └─────────────────────┴────────────────────────────────────────────────────────────────────┘

  One Question

  The ADO PAT currently used for wiki-sync — does it have read access to the code repo (AIE/_git/AIE) too, or just the wiki repo (AIE/_git/AIE.wiki)? They're separate Git repos in ADO. If the PAT is org-scoped it    
  probably covers both, but worth confirming before we build this.
