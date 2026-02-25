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
