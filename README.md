# Sinch-with-Github-actions

A minimal Flask API used to demonstrate how to build a conditional, context-aware email notification pipeline using GitHub Actions and [Sinch Mailgun](https://www.mailgun.com/).

This repo accompanies the article: **"Using Sinch Mailgun to improve the developer experience with GitHub Actions"**.

---

## What this repo does

The app itself is intentionally simple: a two-endpoint Flask API that serves as a realistic stand-in for any production service. The real focus is the
`.github/workflows/` directory, which contains three workflows that cover a common set of notification scenarios teams actually need.

### The three workflows

**1. Production deployment failure alert (`deploy.yml`)**

Triggers on every push to `master` (the `main` branch). Runs the test suite, then simulates a deployment step that fails. When the failure is detected, GitHub Actions immediately sends an HTML email to the on-call engineer with the repo name, branch, commit SHA, the person who triggered the run, and a direct link to
the failed Actions run.

**2. Release summary email (`release.yml`)**

Triggers when a GitHub Release is published. Sends a formatted email to stakeholders containing the version tag, who published it, the full release notes, and a link to the release page on GitHub. No manual announcements needed.

**3. PR merge notification (`pr_merge.yml`)**  

Triggers when a pull request is closed against `master`, but only fires if the PR was actually merged (not just closed). Sends a ping to the PM with the PR
title, the author, who merged it, and a link to the PR.

---

## Project structure

```
Sinch-with-Github-actions/
├── app.py                        # Minimal Flask API (two endpoints)
├── conftest.py                   # Adds project root to Python path for pytest
├── requirements.txt              # Dependencies
├── .gitignore                    # Excludes caches, virtual environments, etc.
├── README.md                     # This file
├── tests/
│   └── test_app.py               # Basic pytest tests for both endpoints
└── .github/
    └── workflows/
        ├── deploy.yml            # Deploy + on-call failure alert
        ├── pr_merge.yml          # PR merge + PM ping
        └── release.yml           # Release publish + stakeholder summary
```

---

## Prerequisites

- A [Sinch Mailgun](https://www.mailgun.com/) account with a verified domain
- A Mailgun API key
- A GitHub repository with Actions enabled

---

## Setup

**1. Clone the repo**
```bash
git clone https://github.com/manishh/Sinch-with-Github-actions
cd Sinch-with-Github-actions
```

**2. Add your secrets to GitHub**  
Go to your repo's **Settings → Secrets and variables → Actions → Secrets** and add the following repository secrets:

| Secret name | Description |
|---|---|
| `MAILGUN_API_KEY` | Your Mailgun API key |
| `MAILGUN_DOMAIN` | Your verified Mailgun sending domain |

**3. Add your email addresses as variables**  
In the same section, switch to the **Variables** tab and add the following:

| Variable name | Description |
|---|---|
| `SENDER_EMAIL` | Sender's email address |
| `ONCALL_EMAIL` | On-call engineer's email address |
| `STAKEHOLDERS_EMAIL` | Stakeholder distribution list or email |
| `PM_EMAIL` | PM's email address |

Using variables (rather than hardcoding emails in the workflow files) allows you to easily update recipients without touching any code or making a new commit.

---

## Triggering the workflows

| Workflow | How to trigger |
|---|---|
| `deploy.yml` | Push any commit to `master` |
| `release.yml` | Publish a new GitHub Release |
| `pr_merge.yml` | Merge a pull request into `master` |

> **Note:** The deploy workflow uses `exit 1` to intentionally simulate a
> deployment failure. This is what triggers the on-call alert. Remove or
> replace this line when adapting the workflow for real use.

---

## Running the app locally

```bash
pip install -r requirements.txt
python app.py
```

The API will be available at `http://localhost:5000`.

```bash
# Test the endpoints
curl http://localhost:5000/
curl http://localhost:5000/health
```

---

## Running the tests

```bash
pytest -v
```

---

## Adapting this for your own project

The workflows in this repo are intentionally minimal. To adapt them:

- Replace the simulated `exit 1` deploy step with your real deployment command
- Add more recipients or use a distribution list for the stakeholder email
- Extend the HTML email body with additional context from the GitHub Actions
  context object (e.g., environment name, test coverage, deployment URL)
- Chain workflows together for multi-stage pipelines

---

**Author:** Manish Hatwalne
