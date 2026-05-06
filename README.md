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

# Debug: Analytics dashboard "By Workflow" tab is empty after deploy                                                  
                                                                                                                        
  ## Symptom                                                                                                            
                                                        
  User deployed all backend changes (Terraform + Lambdas + frontend), but the AdminDashboard's
  Generation Activity → "By Workflow" tab still shows the empty state:

    "No workflow data available yet. This metric started tracking after the latest
     deploy — counts will populate as new generations come in."

  The user has confirmed they generated stories AFTER the deploy, so it's not a "just-deployed,
  zero data" issue.

  ## What's expected

  The "By Workflow" tab should show a bar chart of `data.breakdowns.promptsByWorkflow`,
  populated from the DynamoDB metric `prompts_by_workflow#<storyType>` for each generation
  request type (Decompose Feature, User Story, Technical Enabler, Bug, Spike, Test Case).

  ## Architecture

  Repo: AWS-Lambda-based analytics for an internal story-generation app (story-gen).

  Event flow:
    generate Lambda (lambdas/generate/src/index.js) emits prompt.entered events via
    EventBridge → analytics-aggregator Lambda (lambdas/analytics/src/aggregator.js)
    consumes them and increments DynamoDB metrics in AGGREGATES_TABLE → admin API
    Lambda (lambdas/analytics/src/adminApi.js) reads metrics on dashboard load.

  Region: us-gov-west-1.

  ## Recent changes (3 commits, all deployed)

  Commit 1 (5a3330a) — Terraform: added ANALYTICS_EVENT_BUS_NAME env var to
  ado-integration Lambda. (Not directly relevant to workflow tab; ignore for this issue.)

  Commit 2 (fa9430f) — analytics:
    - lambdas/ado-integration/src/index.js: emits ado.pushed events
    - lambdas/analytics/src/aggregator.js: in handlePromptEntered (~line 199), added:

        const workflow = event.payload?.story_type;
        if (workflow) {
          await incrementMetric(`prompts_by_workflow#${workflow}`);
        }

      Also added handlers for ado.pushed, chat.created, chat.message, intake.submitted.

    - lambdas/analytics/src/adminApi.js: queries 'prompts_by_workflow#' prefix and
      returns it as breakdowns.promptsByWorkflow in the dashboard response.

  Commit 3 (e94bd48) — frontend: AdminDashboard.tsx reads
  data.breakdowns.promptsByWorkflow and renders bars.

  The prompt.entered events ALREADY include story_type in their payload from prior
  versions of the generate Lambda — at lambdas/generate/src/index.js lines 415, 2987,
  3369, 4010. So no new event-emission code was needed; only the aggregator handler.

  ## Investigation checklist

  Please investigate in this order, reporting findings:

  1. **Verify the analytics-aggregator Lambda is running the new code.**
     - Run: aws lambda get-function --function-name <analytics-aggregator-name>
       --region us-gov-west-1 --profile army-story-gen
     - Look at CodeSha256 and LastModified. Confirm LastModified is AFTER the deploy.
     - If not, the deploy didn't actually update the Lambda — re-deploy.

  2. **Check CloudWatch logs for the aggregator Lambda.**
     - Look for log entries like "Processing prompt.entered event" in the last hour.
     - Look for log entries with "Failed to increment metric prompts_by_workflow#..." —
       would indicate IAM or DynamoDB issue.
     - Look for "No handler for event type:" — if you see prompt.entered there, the
       dispatch is broken.

  3. **Check DynamoDB AGGREGATES_TABLE directly.**
     - Run: aws dynamodb query --table-name <AGGREGATES_TABLE>
         --key-condition-expression "pk = :pk AND begins_with(sk, :prefix)"
         --expression-attribute-values '{":pk":{"S":"metric"},":prefix":{"S":"prompts_by_workflow#"}}'
         --region us-gov-west-1 --profile army-story-gen
     - If 0 rows: metric is never being written. Check aggregator code + IAM.
     - If rows exist: data is there; the bug is in adminApi or frontend consumption.

  4. **Check the admin API response directly.**
     - Hit the admin metrics endpoint with auth: GET <api>/admin/metrics
     - Inspect the JSON for breakdowns.promptsByWorkflow. Is it there? Is it []?
       Or undefined?
     - If undefined: adminApi Lambda wasn't redeployed (different Lambda from aggregator).
     - If []: aggregator never wrote metrics; back to step 3.
     - If populated: bug is frontend; check browser network tab for actual response.

  5. **Verify both Lambdas were redeployed.**
     - The repo has SEPARATE Lambdas: analytics-aggregator (event consumer) and
       analytics-admin-api (dashboard read endpoint). Both are in lambdas/analytics/.
       A partial deploy that updated only one would explain this exactly.
     - Check LastModified on both.

  6. **Verify EventBridge rule routes to the updated aggregator.**
     - The rule is in terraform/modules/analytics/main.tf around line 510:
       event_pattern = { source: ["story-gen.backend"], detail-type: ["analytics-event"] }
     - The rule's target is the aggregator Lambda. If a recent terraform apply somehow
       unbound it (unlikely but check), events wouldn't route.

  ## Most likely causes (ranked)

  1. The analytics-admin-api Lambda wasn't redeployed (only the aggregator was).
     The new prompts_by_workflow query lives in adminApi.js, not aggregator.js.
     If admin API runs old code, breakdowns.promptsByWorkflow is undefined →
     frontend shows empty state.

  2. The deploy script (deploy.sh) didn't pick up changes to lambdas/analytics/.
     Check the script logic.

  3. CloudFront/browser cache is serving an old admin metrics response.
     Hard refresh, or check the Network tab in DevTools.

  4. The aggregator Lambda's IAM role lacks dynamodb:UpdateItem on AGGREGATES_TABLE
     for the new key prefix. Unlikely (same table, same operation), but possible if
     there's a key-prefix-scoped policy.

  ## Deliverable

  Report what you find at each step, then identify the root cause and fix.
  Don't apply fixes that "might work" — confirm the actual failure point first
  (e.g., "admin API Lambda was last modified Apr 30, before the deploy — that's
  the issue, redeploy adminApi and the chart should populate within seconds of
  the next dashboard load").

