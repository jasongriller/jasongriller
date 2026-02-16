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



 That matches the existing wiki URL pattern (https://gps-aie-devops@dev.azure.com/gps-aie-devops/AIE/_git/AIE.wiki) — just .wiki vs no .wiki suffix. We're good.

● User approved Claude's plan   
  ⎿  Plan saved to: ~\.claude\plans\sunny-dancing-abelson.md · /plan to edit
     Plan: Auto-Generated AIE Process Document (repo-digest Lambda)                                                                                                                                                        
     Context                                                                                                                                                                                                            
  
     The user wants the AI to have distilled knowledge about how the AIE team works — BA processes, QA workflows, release pipelines — without raw files cluttering the KB. A new scheduled Lambda will clone the
     AIE code repo (not the wiki), read process-relevant files, use Bedrock to generate a structured summary, and save it as a curated KB document that auto-updates daily.

     Key constraint: No new KB tab. The generated doc lives under knowledge-base-docs/context/ in the existing Curated tab.

     ---
     Files to Create (5)

     1. lambdas/repo-digest/package.json

     Dependencies: isomorphic-git, @aws-sdk/client-s3, @aws-sdk/client-secrets-manager, @aws-sdk/client-bedrock-runtime, @aws-sdk/client-bedrock-agent

     2. lambdas/repo-digest/src/index.js (~250-300 lines)

     Handler flow:
     1. Get ADO credentials from Secrets Manager (reuse exact pattern from lambdas/wiki-sync/src/index.js lines 103-117)
     2. Shallow-clone AIE code repo to /tmp/aie-repo using isomorphic-git (reuse pattern from wiki-sync lines 122-175, but with ADO_REPO_URL env var and ADO_REPO_BRANCH for checkout — defaults to
     releases/aie/current)
     3. Get HEAD commit hash via git.resolveRef({ ref: 'HEAD' })
     4. Check existing doc metadata — HeadObjectCommand on knowledge-base-docs/context/aie-team-processes.md to read last-commit-hash from S3 metadata. If same as HEAD → exit early (no changes)
     5. Read relevant files — walk docs/, pipelines/, release-definitions/, data-dictionary/ collecting .md, .yml, .yaml, .json, .txt files
     6. Get git log — git.log({ depth: 50 }) for recent commits, formatted as "date — author — message"
     7. Build Bedrock prompt — System prompt explains the task; user message contains file contents + git log. Includes token budget guard (~200K char limit, prioritize docs/ if over budget)
     8. Call Bedrock ConverseCommand — Same pattern as lambdas/generate/src/index.js line 851: BedrockRuntimeClient + ConverseCommand with system, messages, inferenceConfig
     9. Save to S3 — PutObjectCommand to knowledge-base-docs/context/aie-team-processes.md with metadata { status: 'active', 'last-commit-hash': hash, 'generated-at': timestamp, 'source': 'repo-digest' }
     10. Trigger Bedrock ingestion — StartIngestionJobCommand with curated data source ID (same pattern as wiki-sync lines 332-355)

     Bedrock summarization prompt:

     System: You are an expert technical writer. Given source files from a software repository,
     generate a structured process document that describes how the team operates. Focus on:
     - Release and deployment processes
     - CI/CD pipeline stages and gates
     - BA (Business Analyst) workflows and conventions
     - QA processes and test strategies
     - Development workflow (branching, PR process, code review)
     - Data model and dictionary overview
     Include a "Recent Changes" section at the end summarizing recent commits.
     Output clean, readable markdown with clear section headers.


     3. terraform/modules/repo-digest/main.tf

     Follow exact wiki-sync pattern:
     - null_resource for npm install --production (triggered by filemd5 on package.json + index.js)
     - archive_file for zipping
     - aws_lambda_function — runtime nodejs20.x, timeout 300s, memory 1024MB, ephemeral storage 2048MB (smaller than wiki-sync — we only read select folders, not 10GB)
     - aws_cloudwatch_log_group — 7 day retention
     - aws_cloudwatch_event_rule — cron(30 0 * * ? *) (daily at 00:30 UTC, 30 min after analytics cron)
     - aws_cloudwatch_event_target → Lambda
     - aws_lambda_permission for EventBridge

     Env vars: ADO_REPO_URL, ADO_REPO_BRANCH, S3_BUCKET_NAME, KNOWLEDGE_BASE_ID, DATA_SOURCE_ID (curated), ADO_SECRET_ARN, BEDROCK_MODEL_ID

     4. terraform/modules/repo-digest/variables.tf

     Variables: function_name, resource_prefix, lambda_role_arn, ado_repo_url, ado_repo_branch, s3_bucket_name, knowledge_base_id, data_source_id, ado_secret_arn, bedrock_model_id, aws_region, tags

     5. terraform/modules/repo-digest/outputs.tf

     Outputs: function_name, function_arn

     ---
     Files to Modify (2)

     6. terraform/variables.tf

     Add two new variables:
     variable "ado_repo_url" {
       description = "ADO code repository Git URL"
       type        = string
       default     = "https://gps-aie-devops@dev.azure.com/gps-aie-devops/AIE/_git/AIE"
     }

     variable "ado_repo_branch" {
       description = "Branch to clone from the AIE code repository"
       type        = string
       default     = "releases/aie/current"
     }

     7. terraform/main.tf

     Add new module block after wiki_sync (around line 320):
     module "repo_digest" {
       source = "./modules/repo-digest"

       function_name     = "${local.resource_prefix}-repo-digest"
       resource_prefix   = local.resource_prefix
       lambda_role_arn   = module.iam.lambda_role_arn
       ado_repo_url      = var.ado_repo_url
       ado_repo_branch   = var.ado_repo_branch
       s3_bucket_name    = module.s3.bucket_name
       knowledge_base_id = module.bedrock.knowledge_base_id
       data_source_id    = module.bedrock.data_source_id      # CURATED data source
       ado_secret_arn    = aws_secretsmanager_secret.ado_credentials.arn
       bedrock_model_id  = var.bedrock_model_id
       aws_region        = var.aws_region
       tags              = local.common_tags

       depends_on = [
         module.bedrock,
         module.s3,
         aws_secretsmanager_secret_version.ado_credentials
       ]
     }

     ---
     Key Design Decisions

     ┌────────────────────────────────┬───────────────────────────────────────────────────┬──────────────────────────────────────────────────────────────────────────────────────┐
     │            Decision            │                      Choice                       │                                         Why                                          │
     ├────────────────────────────────┼───────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────┤
     │ New Lambda vs extend wiki-sync │ New Lambda                                        │ Different concern (summarize vs sync), cleaner separation, independent failure       │
     ├────────────────────────────────┼───────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────┤
     │ IAM role                       │ Shared role (module.iam)                          │ Already has bedrock:InvokeModel, S3 read/write, SecretsManager, bedrock ingestion    │
     ├────────────────────────────────┼───────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────┤
     │ Lambda Layer                   │ Not used                                          │ Follows wiki-sync pattern; only needs S3 + Bedrock + SecretsManager + isomorphic-git │
     ├────────────────────────────────┼───────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────┤
     │ Change detection               │ S3 object metadata (last-commit-hash)             │ No new DynamoDB table needed                                                         │
     ├────────────────────────────────┼───────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────┤
     │ S3 path                        │ knowledge-base-docs/context/aie-team-processes.md │ Lands in existing Curated tab, alongside other context docs                          │
     ├────────────────────────────────┼───────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────┤
     │ Schedule                       │ Daily (00:30 UTC)                                 │ Staggered 30 min after analytics cron to avoid overlapping                           │
     ├────────────────────────────────┼───────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────┤
     │ deploy.sh safe?                │ Yes                                               │ Uses s3 sync without --delete, won't remove S3-only docs                             │
     ├────────────────────────────────┼───────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────┤
     │ Bedrock data source            │ Uses existing curated data source                 │ Doc goes under knowledge-base-docs/ prefix, already covered                          │
     └────────────────────────────────┴───────────────────────────────────────────────────┴──────────────────────────────────────────────────────────────────────────────────────┘

     What Won't Change

     - No frontend changes (doc appears in Curated tab automatically)
     - No new DynamoDB tables
     - No new Bedrock data sources
     - No API Gateway changes (Lambda is EventBridge-triggered only)
     - No changes to existing Lambdas

     ---
     Verification

     1. Local check: npm install --production in lambdas/repo-digest/ succeeds
     2. Terraform: terraform validate and terraform plan in terraform/ — should show new Lambda + EventBridge resources
     3. After deploy: Manually invoke the Lambda via AWS Console or CLI to test:
     aws lambda invoke --function-name army-story-gen-dev-repo-digest --region us-gov-west-1 out.json
     4. Verify S3: Check knowledge-base-docs/context/aie-team-processes.md exists in the S3 bucket
     5. Verify KB: Go to Knowledge Base UI → Curated tab → context section → confirm doc appears
     6. Verify RAG: Generate a story and check if the AI references AIE team processes in its output

● Let's go. I'll set up tasks and start building.  




  You don't want raw files. You don't want a new tab. You want the AI to have distilled knowledge about how the AIE team works, and for that knowledge to stay current.

  What I'd Propose: Auto-Generated Curated Doc(s)

  A scheduled Lambda that:

  1. Clones the AIE code repo (same as wiki-sync pattern)
  2. Reads the relevant files — docs/, pipelines/, release-definitions/, data-dictionary/
  3. Uses Bedrock to generate a structured summary — not the raw files, but a human-readable digest like:
    - How the release process works (from pipeline/release-definition files)
    - BA and QA workflows (from docs/)
    - Data model overview (from data-dictionary/)
    - Recent changes — what's been modified, by whom, when (from git log)
  4. Saves it as a curated KB doc under knowledge-base-docs/context/ — shows up in your existing Curated tab, no new tab
  5. Triggers Bedrock ingestion so the AI can reference it during generation
  6. Runs on same 24h schedule — if nothing changed in the repo, doc stays the same

  This way:
  - It lives alongside your other curated docs
  - The AI naturally references AIE team processes when generating stories
  - It stays current without anyone manually updating it
  - No code/YAML polluting your KB — just the distilled "here's how we work" knowledge

  One thing I can't see — what's actually in the docs/ folder of that repo. If it already has well-written process docs, we might not even need Bedrock to summarize — we could just pull those docs directly and add a 
  changelog section.

  Can you tell me (or screenshot) what's in the docs/ folder? That'll tell me whether we need AI summarization or can just pull the docs as-is.
