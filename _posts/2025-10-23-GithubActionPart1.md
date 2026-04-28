---
title: "Hands-On GitHub Actions for Absolute Beginners - I"
date: 2025-10-23
categories: [DevOps & Automation]
tags: [GitHub Actions, CI/CD, Automation]
---

# Hands-On GitHub Actions for Absolute Beginners - I
---
## Getting Your Mind Ready: GitHub Basics
Before diving into automation, let’s quickly review a few key terms in the context of a GitHub repository (a project folder):

- **Issues**: These are like to-do lists or bug reports. When you find a problem or have an idea for a new feature, you create an Issue.
- **Contributors**: Anyone who helps improve the code, often by creating Pull Requests or providing feedback via Issues.

- **Pull Request (PR)** : is how you **ask to add your changes** to someone else’s project.
**Simple flow:**
1. Make changes in your own copy (a **branch**).  
2. Create a **Pull Request (PR)**.  
3. Project maintainers **review** your changes.  
4. If approved, your changes are **merged** into the main project.

---

## Porject Objective
Our goal today is to automatically greet first-time contributors when they open their very first Issue or Pull Request.

---

## Step-by-Step: Creating Your First Action

###  1. Set Up Your Repo

First, you need a place to work.

1. Go to your GitHub account.  
2. Click **New Repository** and name it something simple like `my-first-action`.

---

###  2. Find the Action

GitHub provides many ready-made automation templates.

1. Open your new repo and go to the **Actions** tab.  
2. Look for the section suggesting automation options.  
3. Choose **"Automation / Greeting"** — it greets users who are first-time contributors to your repo.

---

###  3. Configure and Commit

1. Click **Configure** next to the Greeting action.  
2. A new file named **`greetings.yml`** will open — this is your *workflow* file (it tells GitHub what to do and when).  
3. Click **Commit changes** (you can keep the default message).

- GitHub will now create a folder:  
`.github/workflows/`  
and inside it, your workflow file → `greetings.yml`.

###  Understanding the Workflow File (YAML)

The file you just committed is written in **YAML** .  
Here’s a breakdown of the important parts:

```yaml
name: Greetings

on: [pull_request_target, issues]

jobs:
  greeting:
    runs-on: ubuntu-latest
    permissions:
      issues: write
      pull-requests: write
    steps:
    - uses: actions/first-interaction@v1
      with:
        repo-token: ${{ secrets.GITHUB_TOKEN }}
        issue-message: "Hi you just create an issue."
        pr-message: "Message that will be displayed on users' first pull request"
```
###  Key Points Explained

| Code Line                         | Explanation                                                                                  |
|----------------------------------|----------------------------------------------------------------------------------------------|
| `name: Greetings`                 | The  name for your workflow. Shows up in the **Actions** tab.                   |
| `on: [pull_request_target, issues]` | **Trigger**: Tells GitHub when to run this workflow. It runs when a new Pull Request or Issue is opened. |
| `jobs: greeting:`                 | Defines **jobs** in the workflow. Here, we have one job named `greeting`.                  |
| `runs-on: ubuntu-latest`          | Tells GitHub which virtual machine to use. In this case, Ubuntu Linux.                     |
| `permissions: ...`                | Grants the workflow permissions it needs. Here, it can **write** to Issues and Pull Requests. |
| `uses: actions/first-interaction@v1` | Specifies the pre-built GitHub Action that detects a user’s first contribution. Saves you from writing code yourself. |
| `with:`                           | Provides input settings for the action.                                                     |
| `issue-message: "..."`            | The comment posted on a new user’s first Issue.                                             |
| `pr-message: "..."`               | The comment posted on a new user’s first Pull Request.                                      |

###  A Note on `repo-token: secrets GITHUB_TOKEN `

- **`repo-token:`**  
  This is the security credential that gives the Action permission to act on your repository's behalf (for example, posting a comment).

- **` secrets GITHUB_TOKEN `**  
  A special token that GitHub automatically creates for every workflow run.  
  - Temporary secret  
  - Valid only for the duration of the action

---

###  Summary and Knowledge Check

#### Summary

- **GitHub Actions** lets you automate tasks in your repository using **workflows** written in **YAML**.  
- Workflows are triggered by **events**, like opening an Issue or Pull Request.  
- **Actions** run on virtual machines (e.g., `ubuntu-latest`) and use **permissions** defined in the workflow.  
- `repo-token: $ secrets.GITHUB_TOKEN ` provides the **security credentials** needed for the Action to interact with your repository.

---

#### ❓ Knowledge Check

**Question:**  
If your workflow needs permission to post a comment on an Issue, and it uses `repo-token: $ secrets GITHUB_TOKEN `, do you need to manually create the GITHUB_TOKEN secret in your repository settings?
<details>
<summary>💡 Show Answer & Explanation</summary>
Answer:
No. The `GITHUB_TOKEN` is a (temporary, automatic secret) that GitHub provides to every workflow run. You do **not** need to create it manually.


