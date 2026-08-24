Found it. Your secrets have a clear pipeline in this infrastructure — here's the full picture:

  Where secrets live: Vault → sync job → Jenkins credentials

  1. Source of truth: Vault at https://vault.mgmt.digima.com, under the paths:

  - secret/jenkins/text/<credential-id> — secret text (key: text)
  - secret/jenkins/userpass/<credential-id> — username/password (keys: user, pass)
  - secret/jenkins/ssh/<credential-id> — SSH keys (keys: user, key)

  2. The sync job — a Jenkins job called "Jenkins Vault Synchronization" runs jenkins/scripts/SyncCreds/sync_creds.py (via jenkins/Jenkinsfile-SyncCreds). It lists everything under those
  Vault paths and creates/updates a Jenkins global credential whose ID equals the Vault key name (jenkins/scripts/SyncCreds/sync_creds.py:45-49).

  3. Consumption — pipelines reference them by ID via withCredentials(...), and the final resting place in the UI is Manage Jenkins → Credentials → System → Global credentials.

  One convention that affects your Jenkinsfile

  Looking at how existing text secrets are used, text-type values are stored base64-encoded: Jenkinsfile-ServiceTests does echo $env_secret | base64 -di (line 73), and sync_creds.py
  base64-decodes text values when merging privileged secrets (line 40). The sync job copies the value into Jenkins as-is, so the Jenkins credential also holds the base64 string.

  So you have two routes for your two new secrets:

  Route A — Jenkins UI directly (works with the Jenkinsfile as committed): create both credentials by hand with plain values, as described before. Fastest way to get testing today; the
  downside is they'd be the odd ones out, not managed by Vault.

  Route B — follow the Vault convention:
  # generate and store the trigger token
  token=$(openssl rand -hex 20)
  echo "GitHub webhook token param will be: $(printf %s "$token" | base64)"
  vault kv put secret/jenkins/text/ai-review-trigger-token text=$(printf %s "$token" | base64)

  vault kv put secret/jenkins/text/multica-ai-review-webhook-url \
    text=$(printf %s 'https://multica.tailacc92.ts.net/api/webhooks/autopilots/awt_GTcDvC1U_tWKGU0LmMurvmW28lugdLMzOnTs7V9cQD8' | base64)
  then run the Jenkins Vault Synchronization job. Two adjustments follow from the base64 convention: the token you paste into the GitHub webhook URL must be the base64 string (since that's
  what lands in the Jenkins credential and the Generic Webhook Trigger compares raw values), and the Jenkinsfile's curl step needs to decode the URL — I'd change the curl ...
  "$multica_webhook_url" line to "$(echo "$multica_webhook_url" | base64 -di)".

  My recommendation: Route A now to prove the flow end-to-end, migrate to Route B before you merge the PR so it matches team convention. If you go with B, say so and I'll patch the
  Jenkinsfile for the base64 decode.
