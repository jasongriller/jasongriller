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

module.static_website.null_resource.build_and_sync_frontend (local-exec): copy: s3://decomposerai/index.html to s3://decomposerai/utilities/gamification
module.static_website.null_resource.build_and_sync_frontend: Creation complete after 1m38s [id=735259317]
module.static_website.aws_s3_bucket.website (deposed object a96fa702): Destroying... [id=army-story-gen-dev-frontend]
╷
│ Error: deleting S3 Bucket (army-story-gen-dev-frontend): operation error S3: DeleteBucket, https response error StatusCode: 409, RequestID: HHX44HV6QARZQVZV, HostID: 1ufu8SS5gNnusoFJBzDK0MvvWcTeL818I4AhSjJfKesQx19FQzqYz7Twj14pK9iN4ZieohIVU64=, api error BucketNotEmpty: The bucket you tried to delete is not empty
│ 


Found it. Line 374 is the culprit:
hcl
What happened:

Someone on the smallUpdates branch changed bucket_name from "army-story-gen-dev-frontend" to "decomposerai". S3 bucket names are an immutable attribute — changing the name forces Terraform to destroy the old bucket and create a new one (+/-).

So the sequence was:
1. Terraform created the new decomposerai bucket ✓
2. Ran the frontend build + sync into decomposerai ✓  
3. Tried to delete army-story-gen-dev-frontend ✗ — blocked because it's not empty

The good news: army-story-gen-dev-frontend still exists and is untouched. But your Terraform state is now dirty — it has a "deposed" (zombie) reference to the old bucket.

What you need to decide: Was the rename to decomposerai intentional? If not, revert line 374:
hcl
If it was intentional, you need to manually empty and delete army-story-gen-dev-frontend so the next apply can complete cleanly. Either way, you also need to check whether the decomposerai bucket was actually created and whether the app is pointing at it now.
