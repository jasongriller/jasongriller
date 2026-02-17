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



 Regression Test Cases — How Your Team Does It                                                                                                                                                                                                                                                                                                                                                                                                 
  Regression test cases on your team are not about testing whether one feature works. They're about testing whether the whole process still works after someone pushes a change.                                           
  The Core Idea                                                                                                                                                                                                         
  
  Your QA team picks a real business process — like taking a lead record from initial contact all the way through screening, MEPS, and enlistment — and writes a test that walks through that entire journey. Then they
  run that test every time code changes, either on a schedule (nightly) or on-demand through the ADO pipeline (AIE4QA, AIE3SIT environments).

  If someone on Team 6 modifies a validation rule on the Drug Use screening page, the regression test catches whether that change accidentally broke something downstream — like the moral screening completion status, 
  or the data that gets pushed to MIRS.

  What Makes It Different From What You Generate Today

  Your current test cases say: "Given this user story about updating a phone number field, verify the field saves correctly." One screen, one feature, done.

  A regression test case says: "Start with a lead record created from an EMM referral. Take that lead through admin screening (SF-86 sections 22, 23, 24). Verify data persists at each section. Verify the moral       
  screening rollup status updates. Verify AIE pushes the screening result to MIRS. Verify Keystone reflects the updated eligibility. Verify the recruiter's workspace shows the lead is cleared for MEPS scheduling."   

  It's the same lead record, traced through every system it touches.

  The Screening Focus

  Based on that QA slide, the PI8 priority is screenings. That's because the screening flows are where:
  - The most SF-86 sections live (Drug Use, Alcohol, Police Records, Psychological Health, etc.)
  - Multiple teams are building features in parallel (Team 5 does foundation, Team 3 does UI/validation)
  - Data flows across system boundaries (AIE → DISS for background checks, AIE → Keystone for job reservations, AIE → MIRS for processing)
  - A change in one section can break the rollup logic that determines whether an applicant is cleared

  So a regression test case for your team would look something like:

  REG01: Lead Through Moral Screening — Happy Path

  Process: Lead intake → Admin Screening → Drug Use (Section 23) → Alcohol (Section 24) → Police Records (Section 22) → Moral Screening Complete

  Systems: AIE Salesforce, DISS (background check), MIRS

  Phase 1 — Setup: Create lead with basic demographics, assign to recruiter, advance opportunity to screening stage

  Phase 2 — Drug Use E2E: Navigate to Section 23, answer all questions "No", verify section saves, verify section status = Complete

  Phase 3 — Alcohol E2E: Navigate to Section 24, answer all questions "No", verify section saves, verify section status = Complete

  Phase 4 — Police Records E2E: Navigate to Section 22, answer all questions "No", verify section saves, verify section status = Complete

  Phase 5 — Rollup Verification: Verify Moral Screening overall status = Complete, verify no flags raised

  Phase 6 — Interface Checkpoint: Verify screening result data pushed to MIRS, verify downstream systems reflect cleared status

  Variant: REG02 would be the same flow but with "Yes" answers that trigger waiver requirements

  The Provar + ADO Pipeline Connection

  These test cases aren't meant to be run once by a human. They get:
  1. Written as structured steps (which is what your tool would generate)
  2. Imported into Provar via the CSV exports you already support
  3. Automated in Provar against the AIE4QA or AIE3SIT Salesforce orgs
  4. Executed through the ADO pipeline — either on a schedule (every night, every sprint) or on-demand when a team deploys a change
  5. Results feed back into ADO for tracking

  The "AIE Pushes Output to Other Systems" Part

  This is the interface automation piece from the slide. When AIE processes a screening, it doesn't just save data locally — it pushes results to external systems (MIRS, Keystone, DISS, iPERMS, DocuSign). A
  regression test case has to verify those handoffs:

  - After enlistment processing, did AIE send the correct data to Keystone for job reservation?
  - After a scheduled visit is created, did MIRS receive the visit record?
  - After moral screening completes, did the processing result code sync downstream?

  These are the interface checkpoints — the steps in a regression test where you verify data crossed a system boundary correctly.

  TL;DR

  ┌──────────────────┬───────────────────────────────────────────────────────────────────────────────────┐
  │      Aspect      │                           Your Team's Regression Tests                            │
  ├──────────────────┼───────────────────────────────────────────────────────────────────────────────────┤
  │ What             │ End-to-end process flows through AIE, focused on screenings                       │
  ├──────────────────┼───────────────────────────────────────────────────────────────────────────────────┤
  │ Why              │ Catch breaks when any team changes code that touches shared flows                 │
  ├──────────────────┼───────────────────────────────────────────────────────────────────────────────────┤
  │ How traced       │ One record (lead/applicant) followed through every step of a business process     │
  ├──────────────────┼───────────────────────────────────────────────────────────────────────────────────┤
  │ Systems covered  │ AIE + MIRS + Keystone + DISS + DocuSign + iPERMS                                  │
  ├──────────────────┼───────────────────────────────────────────────────────────────────────────────────┤
  │ Tool             │ Provar (automated), run through ADO pipeline                                      │
  ├──────────────────┼───────────────────────────────────────────────────────────────────────────────────┤
  │ When run         │ Scheduled (nightly/sprint) + on-demand after deployments                          │
  ├──────────────────┼───────────────────────────────────────────────────────────────────────────────────┤
  │ Current priority │ Screening flows (SF-86 moral, medical, admin sections)                            │
  ├──────────────────┼───────────────────────────────────────────────────────────────────────────────────┤
  │ Where GenAI fits │ Generate the regression test case steps so QA can import into Provar and automate │
  └──────────────────┴───────────────────────────────────────────────────────────────────────────────────┘

  That's what your QA means when they say they're moving away from basic feature-level test cases toward regression. They don't need "verify this field saves" — they need "trace this applicant through the entire     
  screening pipeline and make sure nothing is broken."




 aws logs tail /aws/lambda/army-story-gen-dev-wiki-sync \                                                                                                                                                              
    --since 1d --follow --profile govcloud --region us-gov-west-1                                                                                                                                                          
                                                                                                                                                                                                                          You'll see output like:                                                                                                                                                                                               

  Upload complete: 3 uploaded, 247 unchanged, 0 failed
  Changes detected: 3 uploaded, 0 deleted — ingestion triggered


  Or if nothing changed:

  Upload complete: 0 uploaded, 250 unchanged, 0 failed
  No changes detected (250 files unchanged) — skipping KB ingestion


  If you want to see the full sync summary at the end:

  aws logs filter-log-events \
    --log-group-name /aws/lambda/army-story-gen-dev-wiki-sync \
    --filter-pattern "Upload complete" \
    --start-time $(date -d '7 days ago' +%s)000 \
    --profile govcloud --region us-gov-west-1
