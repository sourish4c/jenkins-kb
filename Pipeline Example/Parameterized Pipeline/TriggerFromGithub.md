## 🚀 Trigger Jenkins Parameterized Pipeline from Github

Here's a **sample GitHub Actions workflow** that triggers a **Jenkins pipeline build with parameters** whenever there’s a push to the `main` branch.

---

### ✅ GitHub Actions Workflow to Trigger Jenkins Pipeline

This document will cover:

- How to configure GitHub Actions workflow files (.github/workflows/*.yml)
- Methods to securely communicate between GitHub Actions and Jenkins
- Steps to trigger Jenkins jobs remotely using webhooks or API calls
- Potential authentication and security considerations

This is a useful setup when you want to combine GitHub's native CI/CD capabilities with Jenkins' extensive build and deployment features.

---
### 📁 File Location in Your GitHub Repo:

Create this file:

```
.github/workflows/trigger-jenkins.yml
```

### 📄 `trigger-jenkins.yml` Example:

```yaml
name: Trigger Jenkins Build

on:
  push:
    branches:
      - main

jobs:
  trigger-jenkins:
    runs-on: ubuntu-latest

    steps:
      - name: Trigger Jenkins Build
        run: |
          JENKINS_URL="http://jenkins.local:8080"
          JOB_NAME="GreetJob"
          JENKINS_USER="${{ secrets.JENKINS_USER }}"
          JENKINS_TOKEN="${{ secrets.JENKINS_TOKEN }}"

          curl -X POST "$JENKINS_URL/job/$JOB_NAME/buildWithParameters" \
            --user "$JENKINS_USER:$JENKINS_TOKEN" \
            --data-urlencode "NAME=GitHubUser" \
            --data-urlencode "PROJECT=GitHubTriggeredProject"
```

---

### 🔐 Secure Setup: Add Jenkins Credentials to GitHub Secrets

In your **GitHub repository**, go to:

> `Settings → Secrets and variables → Actions → New repository secret`

Add:

* `JENKINS_USER` = your Jenkins username (e.g., `admin`)
* `JENKINS_TOKEN` = your Jenkins API token

---

### ✅ Optional: If Jenkins CSRF Crumb Is Enabled

Update the script block like this:

```yaml
      - name: Trigger Jenkins Build (with Crumb)
        run: |
          JENKINS_URL="http://jenkins.local:8080"
          JOB_NAME="GreetJob"
          JENKINS_USER="${{ secrets.JENKINS_USER }}"
          JENKINS_TOKEN="${{ secrets.JENKINS_TOKEN }}"

          CRUMB=$(curl -s -u "$JENKINS_USER:$JENKINS_TOKEN" "$JENKINS_URL/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,\":\",//crumb)")

          curl -X POST "$JENKINS_URL/job/$JOB_NAME/buildWithParameters" \
            -H "$CRUMB" \
            --user "$JENKINS_USER:$JENKINS_TOKEN" \
            --data-urlencode "NAME=GitHubUser" \
            --data-urlencode "PROJECT=GitHubTriggeredProject"
```

---

### 💡 Customization Tips

* Change the GitHub trigger (`on:` block) to whatever suits your use case (`pull_request`, `workflow_dispatch`, etc.)
* Pass GitHub commit info into Jenkins parameters using `${{ github.actor }}`, `${{ github.repository }}`, etc.
