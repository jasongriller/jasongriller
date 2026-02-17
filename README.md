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



The Problem                                                                                                                                                                                                           
                                                                                                                                                                                                                        
  Our test case generator currently creates feature-level test cases — "does this one field save correctly?" type tests tied to individual user stories. QA has told us they're moving away from that. What they
  actually need are regression test cases that trace an applicant record through the entire process: Screening → Processing → Enlistment → Sustainment → Shipping → Training → Graduation.

  What Regression Test Cases Look Like For Us

  A regression test case takes one applicant (like "Dennis Tester") and walks them through the full pipeline:

  1. Personal Info — fill out name, DOB, contact info, passport, selective service
  2. Screening — complete all screening types (Admin, Moral, Aptitude, Medical, Family & Associates) by filling out the SF-86 sections in Salesforce
  3. Processing — MEPS scheduling, scheduled visits, result codes
  4. Enlistment — contract signing, job reservation, DocuSign
  5. Sustainment — DEP management, Future Soldier training
  6. Shipping → Training → Graduation

  At each stage, verify the data persists, the status updates, and any cross-system handoffs (MIRS, Keystone, DISS) happen correctly.

  The whole point: if someone changes code in one area, does the entire pipeline still work?

  Phased Rollout

  Phase 1 (Now): Creation flows only — regression test cases that cover creating a new applicant record and walking it through the full pipeline from scratch. This is the highest-priority path and where QA needs     
  coverage first.

  Phase 2 (Later): Read, Update, Delete — add regression test cases for viewing existing records, modifying records mid-process, and deletion/cancellation flows (e.g., SV cancellations, opportunity closures, waiver  
  withdrawals).

  Why Our Tool Is the Right Place For This

  - We already have the SF-86 form reference (all 29 sections, every field, every validation rule)
  - We already have the AIE feature inventory documenting all E2E process flows
  - We already export to Provar CSV format, which is what QA uses for automation
  - QA's own PI8 roadmap lists GenAI as a tool they want to leverage for test case generation
  - The infrastructure is built — we just need a new generation mode

  What Changes

  - Add a "Regression Test Case" option alongside the existing Test Case type
  - Instead of pasting a user story, the user describes a process path (e.g., "Full enlistment pipeline" or "Moral screening E2E")
  - Output is a multi-phase test case organized by lifecycle stage, not by individual acceptance criteria
  - Same Provar/ADO CSV exports QA already uses

  Impact

  QA gets regression test cases generated in seconds instead of manually writing them from scratch. The test cases are grounded in our actual SF-86 field reference and AIE process documentation, so they're specific  
  and executable — not generic templates.
