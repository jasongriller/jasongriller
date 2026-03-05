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

Infra Note (Terraform / ADO Setup)

terraform.tfvars is local and gitignored, so it won’t exist when cloning.

Have him create:

terraform/terraform.tfvars

With:

ado_password      = "<Salman's PAT>"
ado_repo_password = "<Jason's PAT>"

Or set the environment variables before deploying.


  Current Setup                                                                                                                                                                                                                                                                                                                                           
  Your wiki-sync Lambda already uses shallow clone (depth: 1) with isomorphic-git. The Lambda has 3 GB memory, 10 GB ephemeral storage, and a 5-minute timeout — so the        constraints are generous.                                                                                                                                                                                                                                                                                                                                 Alternatives Comparison                                                                                                                                                                                                                                                                                                                                   ┌─────────────────────────────────────┬──────────────────────┬──────────┬─────────────────────────────────────────────────────────────────────────────────────────────┐      │              Approach               │      Speed Gain      │  Effort  │                                        Key Trade-off                                        │      ├─────────────────────────────────────┼──────────────────────┼──────────┼─────────────────────────────────────────────────────────────────────────────────────────────┤      │ Git Binary Layer + simple-git       │ 50-80% faster        │ 4-6 hrs  │ Need to build/maintain a Lambda Layer with native git binary for GovCloud                   │      ├─────────────────────────────────────┼──────────────────────┼──────────┼─────────────────────────────────────────────────────────────────────────────────────────────┤      │ Azure DevOps REST API (no git at    │ Variable             │ 10-14    │ Replaces git entirely; fetch wiki pages via /_apis/wiki/wikis/{id}/pages with               │    
  │ all)                                │                      │ hrs      │ recursionLevel=Full                                                                         │
  ├─────────────────────────────────────┼──────────────────────┼──────────┼─────────────────────────────────────────────────────────────────────────────────────────────┤    
  │ Degit                               │ ~80% smaller         │ 2-3 hrs  │ Downloads tarball snapshot only — unknown if ADO supports tarball endpoint                  │    
  │                                     │ download             │          │                                                                                             │    
  ├─────────────────────────────────────┼──────────────────────┼──────────┼─────────────────────────────────────────────────────────────────────────────────────────────┤    
  │ Optimize current isomorphic-git     │ 10-15%               │ 1-2 hrs  │ Add cache param, marginal gains only                                                        │    
  └─────────────────────────────────────┴──────────────────────┴──────────┴─────────────────────────────────────────────────────────────────────────────────────────────┘    

  Top 2 Recommendations

  1. Git Binary Lambda Layer + simple-git (lowest risk, biggest win)

  - Add a Lambda Layer with the native git binary (https://github.com/lambci/git-lambda-layer)
  - Replace isomorphic-git with simple-git (thin wrapper around native git CLI)
  - Enables sparse checkout — only clone the directories you need, skip .attachments/ at the git level
  - 50-80% faster since it's native C instead of pure JavaScript pack-file parsing
  - Caveat: Must verify the binary works in GovCloud Lambda runtime

  2. Azure DevOps REST API (no git dependency at all)

  - Use GET /_apis/wiki/wikis/{wikiId}/pages?recursionLevel=Full&includeContent=True to fetch all pages with content directly
  - Skip git entirely — no clone, no pack files, no .git directory
  - Can do incremental sync (only fetch changed pages)
  - Caveat: More code to write, pagination logic needed, attachment handling unclear

  What about just improving isomorphic-git?

  You could add the cache parameter to reuse intermediate pack-file results between commands, but it's a marginal 10-15% gain. The core issue is that isomorphic-git parses  
  pack files in pure JavaScript — fundamentally slower than native git for large repos.

