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

  MSYS_NO_PATHCONV=1 aws logs tail /aws/lambda/army-story-gen-dev-stories --since 1h --region us-gov-west-1 --profile       
  govcloud --format short | head -40

  Two sanity checks first if that errors again:
  aws configure list-profiles
  aws sts get-caller-identity --profile govcloud --region us-gov-west-1

  AWS_PROFILE=govcloud MSYS_NO_PATHCONV=1 aws logs tail /aws/lambda/army-story-gen-dev-stories --since 1h --region            us-gov-west-1 --format short | head -40     

  jgriller@USLMAJGRILLER1 MINGW64 ~
$ AWS_PROFILE=govcloud MSYS_NO_PATHCONV=1 aws logs tail /aws/lambda/army-story-gen-dev-stories --since 1h --region us-gov-west-1 --format short | head -40
2026-04-23T18:21:14 INIT_START Runtime Version: nodejs:20.v106  Runtime Version ARN: arn:aws-us-gov:lambda:us-gov-west-1::runtime:fde5301768a459759cc7f4cc184b8d3fd09b276d3c42fcb56468f48ed3ebd11a
2026-04-23T18:21:15 START RequestId: f54120be-c497-4f67-b811-b40cdd74da78 Version: $LATEST
2026-04-23T18:21:15 2026-04-23T18:21:15.478Z    f54120be-c497-4f67-b811-b40cdd74da78    INFO    lambda-stories received event: {
  "resource": "/feedback",
  "path": "/feedback",
  "httpMethod": "GET",
  "headers": {
    "accept": "*/*",
    "accept-encoding": "gzip, deflate, br, zstd",
    "accept-language": "en-US,en;q=0.9",
    "Authorization": "Bearer eyJraWQiOiJiV0tzSGJMWllkT09kRHYxbmduVVNEWFdGY0k0MXg3XC9yeEUzWmIyb0NvST0iLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiIwODExNDJjOC05MGMxLTcwNWEtZTQ2Ni03MjY2ZWVkNDNlYzYiLCJpc3MiOiJodHRwczpcL1wvY29nbml0by1pZHAudXMtZ292LXdlc3QtMS5hbWF6b25hd3MuY29tXC91cy1nb3Ytd2VzdC0xX0QxMGVKMGk1OSIsImNvZ25pdG86dXNlcm5hbWUiOiIwODExNDJjOC05MGMxLTcwNWEtZTQ2Ni03MjY2ZWVkNDNlYzYiLCJvcmlnaW5fanRpIjoiZGQwNjg3YTYtNmQ1My00ZTk5LTkyYzgtZDVmZjI5YmE4NjAwIiwiYXVkIjoiNzZnMGQyajVvMTI3dDY3Z2MxZW1iMTU3NnIiLCJldmVudF9pZCI6IjQ0Zjg2OTdjLTkyMWYtNGY5My04NTk1LWI1YzZmNWVkMjA4OCIsInRva2VuX3VzZSI6ImlkIiwiYXV0aF90aW1lIjoxNzc2OTY4MDMxLCJleHAiOjE3NzY5NzE2MzEsImN1c3RvbTpyb2xlIjoiQWRtaW4iLCJpYXQiOjE3NzY5NjgwMzEsImp0aSI6IjA3YzBjYzFkLTRjNDYtNDE4NC1iMjRiLWUyNDY0ZGEwZGU1NSIsImVtYWlsIjoiamdyaWxsZXJAZGVsb2l0dGUuY29tIn0.cyGKWfFPjJmRucLq-bqFYcv1pF1ke7bIwEodcgqp0nEae3f_8DTxmfHWJdMEtr5kFRX5Q_QsI629XYS0bvexcIBbMouKJAtG8Un1ZrQJYMUP705X2YZuWgthNnLrbS447CAKqb4ZYFLj8MAerDIC72xEu6DgZJqubnYyhSnqz86LDgaAPwga3_AZO_2vsZ3jYGHwosLPaICD_CoCW5utlczd92ApyZoCP17u0wNuEnSpCXtiLDFCYYSyfUJTI_HOZKGSDqeGvTsA3Xun7BBQL-WqGrcJ7wrJPzxBNv8MYKzzVbjH91HruknNt3ZGvJrdxak6gEduw_HuozNPa9MpxA",
    "content-type": "application/json",
    "Host": "9fewsfy9bg.execute-api.us-gov-west-1.amazonaws.com",
    "origin": "http://localhost:3000",
    "priority": "u=1, i",
    "referer": "http://localhost:3000/",
    "sec-ch-ua": "\"Google Chrome\";v=\"147\", \"Not.A/Brand\";v=\"8\", \"Chromium\";v=\"147\"",
    "sec-ch-ua-mobile": "?0",
    "sec-ch-ua-platform": "\"Windows\"",
    "sec-fetch-dest": "empty",
    "sec-fetch-mode": "cors",
    "sec-fetch-site": "cross-site",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/147.0.0.0 Safari/537.36",
    "X-Amzn-Trace-Id": "Root=1-69ea631a-5f246c7b69856ae338cb0c69",
    "X-Forwarded-For": "35.145.4.10",
    "X-Forwarded-Port": "443",
    "X-Forwarded-Proto": "https"
  },
  "multiValueHeaders": {
    "accept": [
      "*/*"
    ],
    "accept-encoding": [
      "gzip, deflate, br, zstd"
    ],
    "accept-language": [
      "en-US,en;q=0.9"
    ],
    "Authorization": [
      "Bearer eyJraWQiOiJiV0tzSGJMWllkT09kRHYxbmduVVNEWFdGY0k0MXg3XC9yeEUzWmIyb0NvST0iLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiIwODExNDJjOC05MGMxLTcwNWEtZTQ2Ni03MjY2ZWVkNDNlYzYiLCJpc3MiOiJodHRwczpcL1wvY29nbml0by1pZHAudXMtZ292LXdlc3QtMS5hbWF6b25hd3MuY29tXC91cy1nb3Ytd2VzdC0xX0QxMGVKMGk1OSIsImNvZ25pdG86dXNlcm5hbWUiOiIwODExNDJjOC05MGMxLTcwNWEtZTQ2Ni03MjY2ZWVkNDNlYzYiLCJvcmlnaW5fanRpIjoiZGQwNjg3YTYtNmQ1My00ZTk5LTkyYzgtZDVmZjI5YmE4NjAwIiwiYXVkIjoiNzZnMGQyajVvMTI3dDY3Z2MxZW1iMTU3NnIiLCJldmVudF9pZCI6IjQ0Zjg2OTdjLTkyMWYtNGY5My04NTk1LWI1YzZmNWVkMjA4OCIsInRva2VuX3VzZSI6ImlkIiwiYXV0aF90aW1lIjoxNzc2OTY4MDMxLCJleHAiOjE3NzY5NzE2MzEsImN1c3RvbTpyb2xlIjoiQWRtaW4iLCJpYXQiOjE3NzY5NjgwMzEsImp0aSI6IjA3YzBjYzFkLTRjNDYtNDE4NC1iMjRiLWUyNDY0ZGEwZGU1NSIsImVtYWlsIjoiamdyaWxsZXJAZGVsb2l0dGUuY29tIn0.cyGKWfFPjJmRucLq-bqFYcv1pF1ke7bIwEodcgqp0nEae3f_8DTxmfHWJdMEtr5kFRX5Q_QsI629XYS0bvexcIBbMouKJAtG8Un1ZrQJYMUP705X2YZuWgthNnLrbS447CAKqb4ZYFLj8MAerDIC72xEu6DgZJqubnYyhSnqz86LDgaAPwga3_AZO_2vsZ3jYGHwosLPaICD_CoCW5utlczd92ApyZoCP17u0wNuEnSpCXtiLDFCYYSyfUJTI_HOZKGSDqeGvTsA3Xun7BBQL-WqGrcJ7wrJPzxBNv8MYKzzVbjH91HruknNt3ZGvJrdxak6gEduw_HuozNPa9MpxA"
Exception ignored on flushing sys.stdout:
OSError: [Errno 22] Invalid argument

