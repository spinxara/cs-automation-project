# CS automation case study

A public write-up of **Dani**, an internal customer-support automation service I designed, built, and operate at [wavve Americas](https://www.wavve.com/) (KOCOWA / KOCOWA+).

This repository is a **case study only**. It does not contain source code, configs, prompts, knowledge-base content, or operational data. The production system stays in a private company repository.

**Status:** production service, still expanding which ticket types may auto-reply. Not a finished “full automation” product.

---

## Problem

KOCOWA customer support handles high-volume, multilingual email and web-form tickets: billing, account deletion, playback, subtitles, catalog availability, region access, and how-to questions.

An unconstrained LLM reply is unsafe. A wrong “yes, we have that title,” a guessed App Store vs Stripe cancel path, or a promised account deletion can create chargebacks, bad catalog claims, or privacy issues.

The work needed classification, routing, and *limited* auto-reply — without treating the model as the catalog or billing system of record.

## Role

I specified behavior, implemented the service, operate the production Docker host, and decide policy: which intents may auto-reply vs hand off to a human.

I did **not** build the company’s customer-account API. Dani consumes that service; another engineer authored it.

## What shipped

- End-to-end ticket pipeline: ingest from Zendesk, classify, apply policy, reply or hand off
- Intent taxonomy, classifier, and scope guard, with form-field and screenshot context when the email body is not enough
- Specialized first-email handlers for catalog, subtitles, region, cancel path, subscription visibility, and account deletion
- Ticket routing even when Dani does not send a public reply (form, tags, escalation fields)
- Live, auditable automation policy so operators can turn intents on without a Python deploy
- Flask operator console (overview, metrics, policy toggles, audit history). The browser never holds API keys.
- Process/error logging and internal chat alerts
- Scheduled support-metrics reporting
- Docker Compose for the API, console, and reporting jobs
- Tests around high-risk rules (catalog claims, cancel path, policy gates)

## Stack

Python 3.12, FastAPI, Flask, Pydantic, pytest, MySQL, Docker Compose, Zendesk Support API, OpenAI API.

## Architecture

```mermaid
flowchart LR
  Ticket[Zendesk ticket] --> Webhook[Webhook]
  Webhook --> API[FastAPI service]
  API --> Classify[Classify intent]
  Classify --> Policy[Live policy check]
  Policy --> Lookup[Company systems of record]
  Lookup --> Decision{Auto-reply allowed?}
  Decision -->|yes| Reply[Public reply]
  Decision -->|no| Handoff[Human handoff plus routing]
  API --> Console[Operator console]
  API --> Reports[Scheduled reports]
```

Flow in words:

1. Zendesk sends a ticket payload. The API acknowledges quickly, then waits in production so form fields can finish landing.
2. Idempotent tags prevent webhook retries from double-replying.
3. The service fetches the ticket (including custom fields and optional screenshots), classifies it, and loads the live automation policy.
4. Catalog, entitlement, and billing facts come from company databases and APIs — not from the model.
5. If policy and confidence allow, Dani sends a public reply. Otherwise it hands off with structured notes, tags, and escalation fields.
6. Operators use the console to review coverage and flip intent toggles. Reporting jobs publish metrics on a schedule.

## Engineering decisions

| Problem | What landed |
| --- | --- |
| LLM inventing catalog or billing facts | Classify + tools. Company systems of record answer availability, entitlement, and cancel path. Confidence and policy gates. |
| Webhook retries double-replying | Idempotent processing tags. |
| Title or language only on the form or in a screenshot | Fetch ticket fields and optional image analysis, not a larger webhook blob. |
| Late form fields vs immediate classify | Short processing delay in production (skipped in test). |
| Turning intents off required a deploy | MySQL policy rows + API + console audit history. |
| Reporting hitting Zendesk on every dashboard load | Batch snapshots instead of a write on every ticket. |
| Stripe cancel vs App Store self-serve | Dani never cancels. App stores get self-serve steps; Stripe goes to a human. |
| Chat noise | End-of-run summaries on separate channels from errors and escalations. |
| Draft replies on handoffs | Switchable after CS said they barely used them. |

## Outcomes (honest)

Qualitative, supported:

- Production Docker stack (API, console, reporter) is running.
- From mid-2026 onward, Dani **queues almost all in-scope email/web tickets** — most still hand off because many intents remain off by policy, not because the model failed.
- Auto-reply volume rose as specialized handlers shipped, especially technical-support cases.
- Tickets are **organized even when Dani does not answer**: form routing and escalation fields still land for CS and internal teams.
- Operators can enable or disable sub-intents without shipping Python.

I am **not** claiming CSAT, hours saved, cost savings, deflection rate, or “all tickets auto-reply.” Those were not measured here.

## What this repo is not

- Not the Dani source tree
- Not a runnable demo
- Not official KOCOWA / wavve documentation
- Not a dump of internal hostnames, field IDs, prompts, or customer tickets

See [NOTICE.md](NOTICE.md). Ask before making a private case-study repo public; even architecture write-ups can be confidential.

## Resume bullets

- Built and operate Dani, a production Python/FastAPI service that classifies KOCOWA Zendesk email/web tickets with an LLM, then auto-replies or hands off using confidence gates and a live, auditable MySQL automation policy.
- Implemented first-email automations that query internal catalog and account systems so subtitle, title-availability, region, and cancel-path answers are not model guesses; Stripe cancellations are never executed by the bot.
- Shipped Zendesk form routing, sub-intent tags, and escalation fields so tickets are organized for CS and internal teams even when Dani does not send a public reply.
- Built a Flask operator console (overview, metrics, policy, audit) plus Docker Compose deployment of API, console, and scheduled reporting.

## Connect this folder to GitHub (when you are ready)

This folder is a **new git history**. It has no remotes yet. Create an **empty private** repo on your personal GitHub named `cs-automation-case-study` (do not initialize it with a README), then:

```bash
cd ~/Documents/Personal/Projects/cs-automation-case-study
git remote add origin git@github.com:YOUR_USERNAME/cs-automation-case-study.git
git push -u origin main
```

Keep it private until you (and ideally your manager) review this write-up. Do **not** add a GitHub remote to the company Dani repo on Bitbucket.
