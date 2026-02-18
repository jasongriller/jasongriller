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

  We have a new Lambda (repo-digest) that runs daily on a cron job. It hits the AIE code repo via ADO REST API, reads all the process-relevant files (docs, pipelines,       
  release definitions, data dictionary, manifests, configs), then feeds them to Bedrock which condenses everything into a single markdown doc describing how the team        
  operates — BA processes, QA workflows, CI/CD pipeline stages, release process, branching strategy, etc. That doc gets saved to S3 under our existing Curated KB section and
   auto-triggers a KB ingestion so the AI always has up-to-date knowledge about team processes. It also skips regeneration if nothing in the repo has changed since last run.

   

It's all in the Bedrock prompt at lambdas/repo-digest/src/index.js:248-313. Here's how it works:                                                                                                                                                                                                                                                        
  Step 1 — Collect raw material (fetchRelevantFiles + getRecentCommits):                                                                                                       - Reads every .md, .yml, .yaml, .json, .txt, .xml, .csv file from these 6 folders: docs/, pipelines/, release-definitions/, data-dictionary/, manifest/, config/
  - Caps at 200K chars total (your run hit that limit at 67 files)                                                                                                           
  - Also grabs the last 50 git commits (date, author, message)

  Step 2 — Build the prompt (buildUserPrompt, line 271):
  - Dumps every file as a labeled code block:
  ### /docs/some-file.md
  - Appends the 50 commits as a markdown table

  Step 3 — System prompt tells the AI what to extract (line 248):

  The system prompt says: "You are an expert technical writer. You'll receive source files from the AIE team's repo. Generate a comprehensive markdown document describing   
  how this team operates." Then it lists 7 specific sections to focus on:

  1. Release and Deployment Process
  2. CI/CD Pipeline (stages, gates, automation)
  3. Development Workflow (branching, PRs, code review)
  4. QA Process (testing strategies, quality gates)
  5. BA Process (requirements, story writing, AC conventions)
  6. Data Model & Dictionary
  7. Configuration & Environment

  Key instruction: "Do NOT include raw file contents — synthesize and summarize." So the AI reads pipeline YAML, release definitions, docs, data dictionaries, and distills  
  them into human-readable prose with specific details (pipeline stage names, branch names, environment names, etc.). If a section has no relevant source files, it omits it 
  rather than guessing.

  Step 4 — Bedrock call (generateDigest, line 292):
  - ConverseCommand with temperature: 0.3 (low creativity, factual), maxTokens: 8000
  - Your run produced a 7,971 char digest from 199,741 chars of input — roughly a 25:1 compression ratio

  So basically: dump all the process-relevant files into the AI's context window, tell it "synthesize this into a team processes doc", and it produces one clean markdown    
  file that the KB can then use for RAG retrieval.
