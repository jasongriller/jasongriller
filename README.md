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



● Run these to verify:                                                                                                                                                                                                  
                                                                                                                                                                                                                          1. Manually invoke the Lambda (first run — no cached hash, so it should do a full generate):                                                                                                                          
                                                                                                                                                                                                                          aws lambda invoke --function-name army-story-gen-dev-repo-digest --region us-gov-west-1 out.json && cat out.json
                                                                                                                                                                                                                                                                                                                                                                                                                                                
  2. Check the output: You're looking for "status": "success" with filesProcessed > 0 and a digestLength. If you see "status": "failed", the error message will tell you what went wrong (most likely an auth issue with
   the PAT).

  3. Verify the doc landed in S3:

  aws s3 ls s3://YOUR-BUCKET-NAME/knowledge-base-docs/context/ --region us-gov-west-1


  You should see aie-team-processes.md in the list.

  4. Check CloudWatch logs if anything looks off:

  aws logs tail /aws/lambda/army-story-gen-dev-repo-digest --region us-gov-west-1 --since 10m


  5. Check the KB UI — go to Knowledge Base → Curated tab → look in the context section for the new doc.

  The most likely failure point is the ADO PAT not having read access to the code repo. If you see a git clone auth error, you'll need to update the PAT scope to include the AIE/_git/AIE repo.
