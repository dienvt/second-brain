Idea using multica with github webhook
## Step 1 : Create github app


Using cloudflare to route traffic to local
```
github-hook-app     | --- payload 934b3b92-9b74-11f1-9ce5-f768cf37bae5 (ping) ---
github-hook-app     | {
github-hook-app     |   "zen": "Approachable is better than simple.",
github-hook-app     |   "hook_id": 667624467,
github-hook-app     |   "hook": {
github-hook-app     |     "type": "App",
github-hook-app     |     "id": 667624467,
github-hook-app     |     "name": "web",
github-hook-app     |     "active": true,
github-hook-app     |     "events": [
github-hook-app     |       "issue_comment",
github-hook-app     |       "pull_request",
github-hook-app     |       "pull_request_review",
github-hook-app     |       "pull_request_review_comment",
github-hook-app     |       "pull_request_review_thread"
github-hook-app     |     ],
github-hook-app     |     "config": {
github-hook-app     |       "content_type": "json",
github-hook-app     |       "insecure_ssl": "0",
github-hook-app     |       "secret": "********",
github-hook-app     |       "url": "https://wait-nirvana-evanescence-metres.trycloudflare.com/webhook"
github-hook-app     |     },
github-hook-app     |     "updated_at": "2026-08-19T02:20:41Z",
github-hook-app     |     "created_at": "2026-08-19T02:20:41Z",
github-hook-app     |     "app_id": 4643769,
github-hook-app     |     "deliveries_url": "https://api.github.com/app/hook/deliveries"
github-hook-app     |   }
github-hook-app     | }
```