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

  Recovery steps (run on the deploy machine, in order)

     cd terraform

     # 1. Tell terraform to forget the decomposerai bucket entry (its dependent resources were already destroyed by the failed apply)
     AWS_PROFILE=govcloud terraform state rm module.static_website.aws_s3_bucket.website
     AWS_PROFILE=govcloud terraform state rm module.static_website.aws_s3_bucket_website_configuration.website
     AWS_PROFILE=govcloud terraform state rm module.static_website.aws_s3_bucket_cors_configuration.website
     # (the policy + public_access_block were destroyed in the failed apply, so state probably already lost them; if state rm errors with "not in state" for those, it's fine
      — ignore)
     AWS_PROFILE=govcloud terraform state rm module.static_website.aws_s3_bucket_policy.website 2>/dev/null
     AWS_PROFILE=govcloud terraform state rm module.static_website.aws_s3_bucket_public_access_block.website 2>/dev/null

     # 2. Import the existing army-story-gen-dev-frontend bucket into state at the right address
     AWS_PROFILE=govcloud terraform import module.static_website.aws_s3_bucket.website army-story-gen-dev-frontend

     # 3. Plan — should now show: attach website_configuration + public_access_block + policy + cors to imported bucket, then run null_resource to sync frontend
     AWS_PROFILE=govcloud terraform plan

     # 4. If plan looks right, apply
     bash deploy.sh

     # 5. Verify old URL works: http://army-story-gen-dev-frontend.s3-website-us-gov-west-1.amazonaws.com/

     # 6. Optional cleanup: manually delete the orphaned decomposerai bucket
     AWS_PROFILE=govcloud aws s3 rm s3://decomposerai/ --recursive --region us-gov-west-1
     AWS_PROFILE=govcloud aws s3 rb s3://decomposerai --region us-gov-west-1

     Why this happened + lesson

     The plan file had the exact recovery steps (the prior "Phase 2 — AWS / terraform reconciliation" section) but the deploy machine operator didn't run them — they just
     git pull + bash deploy.sh. Anytime we change a terraform-tracked bucket's name in code while the real bucket already exists at the new name, terraform needs the
     state-rm + import dance done manually before apply. Standard deploys can't handle bucket renames automatically because S3 doesn't support rename.

     Going forward — boss decision pending

     User is asking boss whether to:
     - (A) Continue with army-story-gen-dev-frontend permanently (recover via above steps, no future renames)
     - (B) Re-attempt the decomposerai rename properly (would require yet another careful migration)
     - (C) Set up CloudFront + custom domain (proper fix, no future S3 rename pain)

     Until boss decides, the recovery above gets the app back online on the old URL.

● Plan saved. Nothing for me to execute on this machine — the fix is all on the deploy machine. Here's the tight copy-paste:

  What broke

  You ran bash deploy.sh on the deploy machine. Terraform saw: state has decomposerai, code says army-story-gen-dev-frontend → tried to swap them. Got partway:
  - ✅ Destroyed decomposerai's policy + public_access_block configs
  - ❌ Tried to CREATE army-story-gen-dev-frontend → AWS said "you already own that bucket name" (because it was never deleted — orphaned from this morning)

  Apply halted. Nothing terrible — just stuck mid-swap.

  Run on the deploy machine (in order)

  cd terraform

  # Forget decomposerai (NOT deleted from AWS)
  AWS_PROFILE=govcloud terraform state rm module.static_website.aws_s3_bucket.website
  AWS_PROFILE=govcloud terraform state rm module.static_website.aws_s3_bucket_website_configuration.website
  AWS_PROFILE=govcloud terraform state rm module.static_website.aws_s3_bucket_cors_configuration.website
  AWS_PROFILE=govcloud terraform state rm module.static_website.aws_s3_bucket_policy.website 2>/dev/null
  AWS_PROFILE=govcloud terraform state rm module.static_website.aws_s3_bucket_public_access_block.website 2>/dev/null

  # Bring the OLD bucket back into state
  AWS_PROFILE=govcloud terraform import module.static_website.aws_s3_bucket.website army-story-gen-dev-frontend

  # See what terraform wants to do (expect: attach website config + policy + cors + sync frontend)
  AWS_PROFILE=govcloud terraform plan

  # Apply
  bash deploy.sh

  After that, hit http://army-story-gen-dev-frontend.s3-website-us-gov-west-1.amazonaws.com/ — should be live again.

  Then optionally (whenever)

  # Clean up the orphaned decomposerai bucket
  AWS_PROFILE=govcloud aws s3 rm s3://decomposerai/ --recursive --region us-gov-west-1
  AWS_PROFILE=govcloud aws s3 rb s3://decomposerai --region us-gov-west-1
