# How to use this case-study folder

Notes for you, not for the public README. The portfolio page is [README.md](README.md). Legal baseline is [NOTICE.md](NOTICE.md).

## Connect this folder to GitHub (when you are ready)

This folder is a **new git history**. It has no remotes yet. Create an **empty private** repo on your personal GitHub named `cs-automation-case-study` (do not initialize it with a README), then:

```bash
cd ~/Documents/Personal/Projects/cs-automation-case-study
git remote add origin git@github.com:YOUR_USERNAME/cs-automation-case-study.git
git push -u origin main
```

Keep it private until you (and ideally your manager) review the write-up. Do **not** add a GitHub remote to the company Dani repo on Bitbucket.

## What must stay out of GitHub

- Any file from the company Dani repo (`app/`, `web-console/`, `config/`, `scripts/`, `tests/`, `.env`, SQL, Zendesk trigger JSON)
- Git history, commit messages, or a submodule pointing at the company repo
- Internal names beyond employer + product: hostnames, IPs, Zendesk custom field IDs, DB names, webhook URLs, JWT/embed details, colleague-identifying debug notes
- Customer PII, ticket IDs, screenshots of the live console with real tickets
- KB chunks, prompts, or policy JSON that encode company playbooks
- Exact internal volume tables unless your manager says those numbers may be public

Screenshots: only a dummy/dev screen with fake tickets, and only with approval.

## How to use the write-up

- **Resume:** the four bullets at the bottom of the README. Do not claim CSAT, hours saved, or “I built the account API.”
- **LinkedIn:** 5–8 sentences from Problem, Role, and Outcomes.
- **Interviews:** walk the architecture diagram, then two deep dives (catalog truth + never-cancel Stripe). If they ask for code, say the source is employer-owned.

Dani, KOCOWA, and Zendesk are named on purpose. Internal hostnames, field IDs, prompts, and customer tickets are not.

Ask before making a private case-study repo public; even architecture write-ups can be confidential.
