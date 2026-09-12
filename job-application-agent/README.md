# Job Application Agent

A conservative Python agent for discovering, filtering, scoring, tracking, and preparing job applications. Version 1 is intentionally review-first: it can fill application forms, but it will not press the final Submit button unless explicitly enabled.

## Design goals

- Enforce hard constraints before scoring.
- Never invent experience, certifications, education, metrics, compensation history, or references.
- Treat ambiguous or consequential questions as `REQUIRES_REVIEW`.
- Deduplicate jobs and keep an application ledger.
- Prefer direct employer application URLs.
- Support browser-assisted form filling with Playwright.
- Default to `review_before_submit: true`.

## Quick start

```bash
cd job-application-agent
python -m venv .venv
source .venv/bin/activate
pip install -e .
playwright install chromium
cp config/profile.example.yaml config/profile.yaml
cp config/policy.example.yaml config/policy.yaml
python -m job_agent.cli init-db
python -m job_agent.cli score examples/job.json
python -m job_agent.cli apply examples/job.json --dry-run
```

`config/profile.yaml`, `config/policy.yaml`, resumes, local databases, and browser session data are git-ignored. Keep private candidate data out of source control.

## Workflow

1. A job source produces a normalized `Job` record.
2. `PolicyEngine` rejects hard conflicts and flags unknowns.
3. `Scorer` assigns a 0–100 fit score using title/seniority, networking, automation, leadership, AI/agentic relevance, compensation, and work arrangement.
4. `Ledger` prevents duplicate applications.
5. `AnswerEngine` maps form questions to verified profile facts and returns `REQUIRES_REVIEW` for anything unknown or consequential.
6. `BrowserApplier` opens the application, fills safe fields, uploads the selected resume, and pauses before submission by default.

## Safety model

The agent has three answer states:

- `ANSWER`: verified fact can be entered automatically.
- `DECLINE`: use a configured non-disclosure answer when lawful and appropriate.
- `REQUIRES_REVIEW`: the candidate must decide.

The browser layer refuses to submit when unresolved review items exist.

## ATS support

The browser layer uses label/name/placeholder heuristics rather than site-specific brittle selectors. Adapter modules can later be added for Greenhouse, Lever, Ashby, Workday, and other ATS platforms.

## Important

This project automates repetitive application work; it does not bypass CAPTCHAs, impersonate another person, evade employer controls, or fabricate qualifications.