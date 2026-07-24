### How the pieces fit

Jenkins doesn't need a special Claude plugin. Claude Code ships as an npm package and runs non-interactively with the `-p` (print/headless) flag — the `-p` flag turns Claude Code into a standard Unix command, compatible with pipes, redirections, and shell scripts. So the flow is: [SFEIR Institute](https://institute.sfeir.com/en/claude-code/claude-code-headless-mode-and-ci-cd/)

1. **PR opened** → your Git host fires a webhook → Jenkins job triggers
2. Jenkins **checks out** the PR branch and computes the diff
3. Jenkins **installs Claude Code** (`npm i -g @anthropic-ai/claude-code`) and runs `claude -p` with your prompt + diff. The API key comes from Jenkins credentials as `ANTHROPIC_API_KEY`.
4. Claude **automatically loads** your skills and MCP servers from files committed in the repo
5. Jenkins **posts the output back** as a PR comment via your host's CLI (`gh`, `glab`, or the Bitbucket API)

### Where skills and MCP live in git

This is the part you specifically asked about, and it's clean — both are just files you commit:

```
your-repo/
├── Jenkinsfile
├── CLAUDE.md                 # repo conventions + review standards (always loaded)
├── .mcp.json                 # project-scope MCP servers (NO secrets)
└── .claude/
    └── skills/
        └── pr-review/
            └── SKILL.md       # the review playbook
```

For skills: commit `.claude/skills/` to version control — they're versioned with the codebase, new hires get them on first clone, and skill changes go through code review like everything else. Each skill is a directory containing a SKILL.md file that starts with YAML frontmatter with required metadata: name and description. [Duet](https://duet.so/guides/claude-code-skills-complete-guide)[Anthropic](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

For MCP: `.mcp.json` is your project-level MCP config — put it in the repo root and commit it if it contains only non-secret configuration. Keep tokens out of it and use env vars instead. Project scope means the config is shared with the repo, so anyone (including Jenkins) running Claude from the repo root picks it up. [LLMx](https://llmx.tech/blog/definitive-guide-to-claude-code-setup-claude-md-mcps-skills/)[Okhlopkov](https://okhlopkov.com/claude-code-setup-mcp-hooks-skills-2026/)

A minimal `.mcp.json` — note the secret is referenced, not embedded:

json

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "${GITHUB_TOKEN}" }
    }
  }
}
```

A minimal `SKILL.md`:

markdown

```markdown
---
name: pr-review
description: Review a pull request diff for bugs, security issues, and violations of our conventions. Use whenever reviewing changed code in CI.
---

When reviewing a PR diff:
1. Check against the conventions in CLAUDE.md.
2. Flag security issues (injection, secrets, auth gaps) as 🔴 blocking.
3. Flag likely bugs and missing tests as 🟡.
4. Skip pure style nits handled by linters.
5. Output a markdown summary grouped by severity. If nothing is wrong, say so briefly.
```

### The Jenkinsfile

This assumes a GitHub repo + the `gh` CLI. I've noted the GitLab/Bitbucket swaps below.

groovy

```groovy
pipeline {
  agent { docker { image 'node:22' } }   // Node 22 = Claude Code requirement

  environment {
    ANTHROPIC_API_KEY = credentials('anthropic-api-key')  // Jenkins credential
    GH_TOKEN          = credentials('github-token')
    GITHUB_TOKEN      = credentials('github-token')        // used by .mcp.json
  }

  stages {
    stage('Install') {
      steps {
        sh 'npm install -g @anthropic-ai/claude-code'
        sh 'apt-get update && apt-get install -y gh'   // GitHub CLI
      }
    }

    stage('Review') {
      steps {
        sh '''
          # Diff of the PR against the base branch
          git fetch origin ${CHANGE_TARGET}
          git diff origin/${CHANGE_TARGET}...HEAD > pr.diff

          claude -p "Review the changes in pr.diff using the pr-review skill. \
                     Output a concise markdown review." \
            --mcp-config .mcp.json --strict-mcp-config \
            --permission-mode acceptEdits \
            --allowedTools "Read,Bash" \
            --max-turns 4 \
            --max-budget-usd 1.00 \
            --model sonnet \
            --output-format text > review.md
        '''
      }
    }

    stage('Post comment') {
      steps {
        sh 'gh pr comment ${CHANGE_ID} --body-file review.md'
      }
    }
  }
}
```

A few flags matter a lot in CI:

- `--mcp-config .mcp.json --strict-mcp-config` — loads exactly your repo's servers and nothing else. This matters because `--strict-mcp-config` ensures only the specified servers are loaded, ignoring any from `.mcp.json` or user settings; `--mcp-config` is useful in CI/CD where you need consistent MCP server configuration across runs. It also sidesteps the interactive trust prompt that project MCP servers normally require. [ComputingForGeeks](https://computingforgeeks.com/claude-code-cheat-sheet/)
- `--allowedTools` / a non-interactive permission mode — without this, the job can hang waiting for approval. Always restrict allowed tools in CI workflows to guarantee execution security and predictability. [SFEIR Institute](https://institute.sfeir.com/en/claude-code/claude-code-headless-mode-and-ci-cd/command-reference/)
- `--max-turns` and `--max-budget-usd` — hard caps so a runaway prompt can't drain your budget. A standard diff review rarely needs more than 2 turns. Realistically, an automated code review on a 500-line diff takes 20–45 seconds and costs roughly $0.03 in API tokens. [SFEIR Institute](https://institute.sfeir.com/en/claude-code/claude-code-headless-mode-and-ci-cd/command-reference/)[SFEIR Institute](https://institute.sfeir.com/en/claude-code/claude-code-headless-mode-and-ci-cd/)

### Triggering on PR creation

Jenkins doesn't poll for PRs natively — you wire the webhook:

- **GitHub** — Use a Multibranch Pipeline job (it auto-discovers PRs), or the "GitHub Pull Request Builder" plugin. With Multibranch, the `CHANGE_ID`, `CHANGE_TARGET` env vars above are populated automatically for PR builds. Point a GitHub webhook at `https://<jenkins>/github-webhook/`.
- **GitLab** — "GitLab" plugin, trigger on Merge Request events; post back with `glab mr note`.
- **Bitbucket** — "Bitbucket Branch Source" plugin; post back via the REST API with curl.

### A couple of things to get right

Put the actual review standards in `CLAUDE.md` since CLAUDE.md is always loaded while a skill loads only when needed — keep CLAUDE.md short (repo truth and conventions) and put the step-by-step review procedure in the skill. Store the API key as a Jenkins credential, never in the repo. And run the job in an ephemeral container (the `docker` agent above) so each review is isolated. [Duet](https://duet.so/guides/claude-code-skills-complete-guide)

One open question that changes the trigger and comment-posting steps: which Git host are you on — GitHub, GitLab, or Bitbucket? If you tell me, I can tighten the Jenkinsfile to exactly your setup.





## You said: What if I want to create just like a bot then if I assign that bot to the PR the code review will be trigger?

What if I want to create just like a bot then if I assign that bot to the PR the code review will be trigger?

Jun 18





## Claude responded: Yes, that's a cleaner UX and it's very doable.

Architected webhook-triggered bot review system using machine user identity

Architected webhook-triggered bot review system using machine user identity

Yes, that's a cleaner UX and it's very doable. The trick is that "assigning a bot as reviewer" is just a webhook event you can react to. When you request a review from a user, GitHub fires a `pull_request` webhook with `action: review_requested` and a `requested_reviewer` field. So you make a bot account, assign it, and have Jenkins trigger only when the requested reviewer is that bot.

There are three parts: the bot identity, a payload-conditional trigger, and posting the result back as the bot.

### 1. The bot identity

Create a dedicated **machine user account** on your Git host — e.g. a GitHub login like `claude-review-bot`. You assign reviews to _users and teams_, so a normal account is what gives you the "assign it to the PR" interaction. (A GitHub App has its own identity and finer permissions, but Apps aren't requestable as reviewers, so for this exact UX a machine user is simplest.)

Give that account read access to the repo, generate a personal access token for it, and store the token in Jenkins credentials. When Jenkins posts the review using _that_ token, the comment shows up as coming from the bot.

### 2. Trigger only when the bot is requested

The Multibranch setup from before discovers PRs but doesn't let you branch on _which_ webhook action fired. For "react to review-request from a specific user," use the **Generic Webhook Trigger** plugin — it extracts JSON fields from the payload and only runs the build when they match a filter:

groovy

```groovy
properties([
  pipelineTriggers([
    [$class: 'GenericTrigger',
      genericVariables: [
        [key: 'action',      value: '$.action'],
        [key: 'reviewer',    value: '$.requested_reviewer.login'],
        [key: 'pr_number',   value: '$.number'],
        [key: 'pr_branch',   value: '$.pull_request.head.ref'],
        [key: 'base_branch', value: '$.pull_request.base.ref'],
        [key: 'repo',        value: '$.repository.full_name']
      ],
      token: 'claude-pr-review',
      causeString: 'Review requested from bot on PR #$pr_number',
      // only fire when the bot is the one being asked to review
      regexpFilterText: '$action::$reviewer',
      regexpFilterExpression: '^review_requested::claude-review-bot$'
    ]
  ])
])
```

Then point a GitHub webhook for **Pull request** events at:  
`https://<jenkins>/generic-webhook-trigger/invoke?token=claude-pr-review`

A nice side effect: every time someone _re-requests_ the bot's review (the little ↻ next to its name), the event fires again — so "re-review after I push fixes" works for free.

### 3. The pipeline body

Same Claude invocation as before, just using the variables pulled from the payload, and posting as the bot:

groovy

```groovy
stage('Review') {
  steps {
    sh '''
      git fetch origin ${base_branch} ${pr_branch}
      git checkout ${pr_branch}
      git diff origin/${base_branch}...HEAD > pr.diff

      claude -p "Review pr.diff using the pr-review skill. Output concise markdown." \
        --mcp-config .mcp.json --strict-mcp-config \
        --permission-mode acceptEdits --allowedTools "Read,Bash" \
        --max-turns 4 --max-budget-usd 1.00 --model sonnet \
        --output-format text > review.md
    '''
  }
}
stage('Post as bot') {
  steps {
    // GH_TOKEN = the BOT's token, so the comment is authored by the bot
    sh 'gh pr comment ${pr_number} --repo ${repo} --body-file review.md'
  }
}
```

If you'd rather the bot leave a _formal_ review (with an Approve / Request-changes state) instead of a plain comment, swap the last line for `gh pr review ${pr_number} --comment --body-file review.md`. Your skills and `.mcp.json` still load automatically from the repo exactly as before — nothing changes there.

### Two lighter alternatives

If managing a machine-user seat is annoying, these give a similar "summon the bot" feel with less setup:

- **Label trigger** — react to `action: labeled` where the label is `claude-review`. No extra account needed; you just slap a label on the PR. Same Generic Webhook Trigger, filter on `^labeled::claude-review$` reading `$.label.name`.
- **Comment/mention trigger** — react to `issue_comment` containing `@claude review`. This is the pattern Anthropic's own GitHub Action uses, where it auto-detects an `@claude` mention versus an automation prompt — but on Jenkins you'd just filter the comment body yourself.

The reviewer-assignment approach you described is the most "native" feeling of the three. The label one is usually the least friction if you don't want to pay for or manage a bot account. Which Git host are you on? The webhook field paths differ slightly on GitLab and Bitbucket, and I can give you the exact JSONPath for whichever you're using.