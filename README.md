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

https://teams.microsoft.com/meet/33123720015122?p=2YhFCjuMMw8UxLzkqI


Sprint Notes Generator — Analytics Tracking Proposal
AI Team · Prepared March 2025 · Internal Use Only
1. Purpose
The Sprint Notes Generator is actively used by the Training Enablement team to convert ADO work items into structured sprint notes. This brief proposes adding usage analytics to the Admin Dashboard so leadership can track adoption, quality, and areas for improvement — without disrupting the user experience.

2. What We Already Track
Each successful generation already emits a detailed analytics event with 25+ data points. However, the Admin Dashboard doesn't surface any of it today.

Category	Data Available	Surfaced in Dashboard?
Generation count (total & daily)	Stored per day and all-time	Stored, not displayed
Per-user generation count	Per-user counter in DynamoDB	Stored, not displayed
Output mode (auto / enhancements / integrations / both)	In event payload	Not aggregated
Quality gate pass/fail	In event payload	Not aggregated
Deep review usage	In event payload (enabled & applied)	Not aggregated
Row counts (enhancements, integrations)	In event payload	Not aggregated
Generation attempts & runtime	In event payload	Not aggregated
Exports (clipboard, CSV, markdown)	Tracked via frontend events	Yes — total only
Generation failures / cancellations	—	Not tracked
Input method (paste vs. file upload)	—	Not tracked
Feature team name	—	Not tracked
User edits to generated output	—	Not tracked
3. Recommended Approach
Strategy: Unlock existing data first, then add targeted new events
90% of the insight is already in the event payload — we just need the aggregator to break it down and the dashboard to display it. Only one new backend event is needed for failure tracking.

Step A — Parse existing payload in the aggregator (backend only)
Update the analytics aggregator to extract additional counters from the existing sprint-notes.generated event:

Generations by output mode (auto / enhancements / integrations / both)
Deep review enabled vs. applied counts
Quality gate pass vs. fail counts
Average enhancement rows per generation
Generations by feature team name (when provided)
Step B — Add one new event for failures
Emit a sprint-notes.generation-failed event when generation errors out. This enables tracking success rate over time — a key reliability metric.

Step C — Add Sprint Notes section to Admin Dashboard
Widget	Data Source	What It Shows
Sprint Notes KPI Card	Existing counters	Total generations, generations today, success rate
Generations Per Day (line chart)	Existing daily counters	Usage trend over last 120 days
Output Mode Distribution	New aggregates (Step A)	Which modes are most used
Deep Review Adoption	New aggregates (Step A)	% of generations using quality review
Quality Gate Pass Rate	New aggregates (Step A)	How often the AI produces passing output
Top Teams	New aggregates (Step A)	Which feature teams use the tool most
4. Effort Estimate
Step	Scope	Files Changed
A — Aggregator updates	Backend only	aggregator.js, adminApi.js
B — Failure event	Backend only	sprint-notes/index.js, aggregator.js
C — Dashboard UI	Frontend only	AdminDashboard.tsx, api.ts
No new infrastructure (DynamoDB tables, EventBridge rules, or IAM policies) required — all data flows through existing pipelines.

5. Future Possibilities
Once the foundation is in place, these can be added incrementally:

Input method tracking — Know whether paste or file upload produces better results
Edit rate tracking — Measure how much users modify AI output (quality signal)
Recruiting stage accuracy — Track how often users override AI-assigned stages
Time-to-export — Measure how long users review before copying
Repeat generation rate — Detect when users retry, indicating unsatisfactory output
