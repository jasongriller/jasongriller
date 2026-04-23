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

 Runtime.ImportModuleError: Error: Cannot find module 'uuid'
  Require stack:
  - /var/task/src/index.js
  - /var/runtime/index.mjs

  Every cold start after 18:25 (when layer v16 was published) fails with the same error. Earlier invocations (18:21,        
  pre-deploy, runtime v106) worked fine. Post-deploy (18:32, runtime v112), every single one errors out at init because uuid
   is missing from the published layer.

  arn:aws-us-gov:lambda:us-gov-west-1:456072988267:layer:army-story-gen-dev-shared-layer:17
