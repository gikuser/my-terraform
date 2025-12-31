# GitHub Actions Comprehensive Cheatsheet (End-to-End CI/CD)

This cheatsheet explains **GitHub Actions from zero to production**, including:

* How code moves from the repository to the runner
* Workflow & YAML syntax
* Secrets, artifacts, caching
* Notifications (email/Slack)
* Deploying to **AWS EC2** (copy, build, run)
* Security & best practices

All sections include **official documentation links** for deeper analysis.

---

## 1. What GitHub Actions Is

GitHub Actions is a **CI/CD automation platform** built into GitHub.

You define workflows that:

* Trigger on events (push, PR, schedule, manual)
* Run jobs on runners (GitHub-hosted or self-hosted)
* Execute steps (shell commands or actions)

Docs: [https://docs.github.com/en/actions](https://docs.github.com/en/actions)

---

## 2. How Code Flows (Mental Model)

```text
GitHub Repo
   ↓ (event: push / PR)
GitHub Actions Workflow
   ↓
Runner VM (Ubuntu/Windows/macOS)
   ↓
Code is checked out into filesystem
   ↓
Steps run (build, test, deploy)
```

Key point:

> **Nothing exists on the runner until you explicitly check it out.**

---

## 3. Workflow File Structure

Workflows live here:

```text
.github/workflows/ci.yml
```

Basic skeleton:

```yaml
name: CI Pipeline

on:
  push:
    branches: ["main"]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Hello World"
```

Workflow Docs: [https://docs.github.com/en/actions/using-workflows](https://docs.github.com/en/actions/using-workflows)

---

## 4. Triggers (`on`)

```yaml
on:
  push:
  pull_request:
  workflow_dispatch:
  schedule:
    - cron: "0 0 * * *"
```

Trigger Docs: [https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)

---

## 5. Jobs

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
```

### Job Dependencies

```yaml
jobs:
  build:
  deploy:
    needs: build
```

Jobs Docs: [https://docs.github.com/en/actions/using-jobs](https://docs.github.com/en/actions/using-jobs)

---

## 6. Runners

### GitHub-hosted

```yaml
runs-on: ubuntu-latest
```

### Self-hosted (e.g., EC2 runner)

```yaml
runs-on: self-hosted
```

Runner Docs: [https://docs.github.com/en/actions/hosting-your-own-runners](https://docs.github.com/en/actions/hosting-your-own-runners)

---

## 7. Steps

### Shell Step

```yaml
- name: Install deps
  run: |
    npm install
    npm test
```

### Action Step

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 20
```

Steps Docs: [https://docs.github.com/en/actions/using-jobs/using-steps-in-a-job](https://docs.github.com/en/actions/using-jobs/using-steps-in-a-job)

---

## 8. Copying Code to the Runner

```yaml
- uses: actions/checkout@v4
```

This:

* Clones the repository
* Checks out the commit that triggered the workflow

Docs: [https://github.com/actions/checkout](https://github.com/actions/checkout)

---

## 9. Environment Variables

### Inline

```yaml
env:
  APP_ENV: production
```

### Step-level

```yaml
- run: echo $APP_ENV
```

Env Docs: [https://docs.github.com/en/actions/learn-github-actions/environment-variables](https://docs.github.com/en/actions/learn-github-actions/environment-variables)

---

## 10. Secrets (Critical)

### Define

* Repo → Settings → Secrets and variables → Actions

### Use

```yaml
- run: echo "$AWS_ACCESS_KEY_ID"
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
```

Secrets Docs: [https://docs.github.com/en/actions/security-guides/encrypted-secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

---

## 11. Artifacts (Pass Files Between Jobs)

### Upload

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: build
    path: dist/
```

### Download

```yaml
- uses: actions/download-artifact@v4
```

Docs: [https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts)

---

## 12. Caching (Speed Up Builds)

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: npm-${{ hashFiles('package-lock.json') }}
```

Cache Docs: [https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)

---

## 13. Deploying to AWS EC2 (SSH Method)

### Required

* EC2 instance
* Security group allows SSH (22)
* Private key stored as GitHub secret

### Copy Code to EC2

```yaml
- name: Copy files
  uses: appleboy/scp-action@v0.1.7
  with:
    host: ${{ secrets.EC2_HOST }}
    username: ec2-user
    key: ${{ secrets.EC2_SSH_KEY }}
    source: "."
    target: "/home/ec2-user/app"
```

### Run Commands on EC2

```yaml
- name: Deploy
  uses: appleboy/ssh-action@v1.0.3
  with:
    host: ${{ secrets.EC2_HOST }}
    username: ec2-user
    key: ${{ secrets.EC2_SSH_KEY }}
    script: |
      cd app
      docker build -t myapp .
      docker run -d -p 80:80 myapp
```

Docs:

* [https://github.com/appleboy/ssh-action](https://github.com/appleboy/ssh-action)
* [https://github.com/appleboy/scp-action](https://github.com/appleboy/scp-action)

---

## 14. Deploying via AWS CLI (Alternative)

```yaml
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::123456789:role/GitHubRole
    aws-region: us-east-1
```

Docs: [https://github.com/aws-actions/configure-aws-credentials](https://github.com/aws-actions/configure-aws-credentials)

---

## 15. Email Notifications

### Using SMTP (Example)

```yaml
- uses: dawidd6/action-send-mail@v3
  with:
    server_address: smtp.gmail.com
    server_port: 465
    username: ${{ secrets.EMAIL_USER }}
    password: ${{ secrets.EMAIL_PASS }}
    to: admin@example.com
    subject: Build Status
    body: Workflow completed
```

Docs: [https://github.com/dawidd6/action-send-mail](https://github.com/dawidd6/action-send-mail)

---

## 16. Conditional Execution

```yaml
- name: Only on main
  if: github.ref == 'refs/heads/main'
```

Condition Docs: [https://docs.github.com/en/actions/learn-github-actions/expressions](https://docs.github.com/en/actions/learn-github-actions/expressions)

---

## 17. Matrix Builds

```yaml
strategy:
  matrix:
    node: [18, 20]
```

Matrix Docs: [https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs)

---

## 18. Permissions (Security)

```yaml
permissions:
  contents: read
  id-token: write
```

Permissions Docs: [https://docs.github.com/en/actions/security-guides/automatic-token-authentication](https://docs.github.com/en/actions/security-guides/automatic-token-authentication)

---

## 19. Debugging

```yaml
- run: env
- run: ls -la
```

Enable debug logs:

```text
ACTIONS_STEP_DEBUG=true
```

---

## 20. Best Practices

* Use **OIDC instead of AWS keys**
* Pin action versions
* Use least privilege permissions
* Keep workflows small
* Separate build & deploy jobs

---

## 21. Official References (Bookmark)

* GitHub Actions Docs: [https://docs.github.com/en/actions](https://docs.github.com/en/actions)
* Actions Marketplace: [https://github.com/marketplace?type=actions](https://github.com/marketplace?type=actions)
* Workflow Syntax: [https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

---

## 22. Typical End-to-End Flow

```text
Push code → CI tests → Build → Upload artifact → Deploy to EC2 → Notify
```

---

## 23. Next Advanced Topics

* Self-hosted runners on EC2
* ECS / EKS deployments
* Terraform + GitHub Actions
* Blue/Green deployments
* Rollbacks & health checks

(Ask for a **ready-to-run production pipeline** if you want.)
