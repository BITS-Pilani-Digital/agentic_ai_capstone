# ThreadWise AI Learning Companion

A four-week Agentic AI capstone: a learning companion that helps people understand reviewed learning material, practise, rehearse short interviews and plan study time.

One reusable application serves different subjects through **reviewed learning packs**. The pilot uses one workspace and two small packs. Every factual answer cites its source, and saved plans and progress change **only after explicit learner approval**.

> **Status: bootstrap only.** This repository contains documentation and configuration templates. No application code has been written and no AWS resources have been provisioned.

## Modes

| Mode | What it does | Limit |
|---|---|---|
| **Learn** | Cited explanations at the learner's chosen level | Unsupported questions get an honest abstention |
| **Practice** | MCQ and short-answer revision questions with cited feedback | At most **5** questions per set |
| **Interview** | Text-only interview with evidence-based feedback | **3** main questions and at most **1** follow-up |
| **Plan** | Study plan within the learner's time budget | Saved only after learner approval |

## Technology

- Python, Streamlit, LangGraph, PostgreSQL with pgvector, MCP, Docker Compose
- Generation through Groq with `openai/gpt-oss-20b` (free tier; **no paid fallback**, no SageMaker)
- Local embeddings with `sentence-transformers/all-MiniLM-L6-v2`
- Development in GitHub Codespaces; deployment on AWS Lightsail with S3 and Cognito

Full constraints are in [docs/requirements.md](docs/requirements.md).

## Documentation

| Document | Purpose |
|---|---|
| [docs/README.md](docs/README.md) | Index and status of the Version 7 source documents |
| [docs/requirements.md](docs/requirements.md) | Requirements, evaluation and release criteria, open decisions, preparation |
| [docs/architecture.md](docs/architecture.md) | Architecture and data model |
| [docs/backlog.md](docs/backlog.md) | Cumulative weekly tasks with acceptance criteria |
| [docs/branching.md](docs/branching.md) | Branch and tag strategy |
| [docs/codespaces.md](docs/codespaces.md) | Creating and rebuilding a Codespace |
| [.github/copilot-instructions.md](.github/copilot-instructions.md) | Rules for AI-assisted development |

The two Version 7 source documents belong in `docs/` but may still be missing; see [docs/README.md](docs/README.md).

## Setup sequence

Complete the steps in order. Stop at the first step that fails and fix it before continuing.

1. **Confirm the entry criteria.** Review the preparation items (`PREP-01` to `PREP-11`) in [docs/backlog.md](docs/backlog.md) and the open decisions (`ISS-xx`) in [docs/requirements.md](docs/requirements.md). Week 1 should not start until they are met or an explicit reduced scope is agreed.
2. **Add the Version 7 documents to `docs/`** (see [docs/README.md](docs/README.md)).
3. **Connect this folder to the project repository** (`https://github.com/BITS-Pilani-Digital/agentic_ai_capstone`). It is currently **public**, so treat everything committed as visible to anyone.
   ```bash
   git init -b main
   git add .
   git status                       # check that no .env or other secrets are listed
   git commit -m "Bootstrap repository documentation and configuration"
   git remote add origin https://github.com/BITS-Pilani-Digital/agentic_ai_capstone.git
   git push -u origin main
   ```
4. **Create the branches** as described in [docs/branching.md](docs/branching.md). Only `week-01` is needed to start; create each later branch when the previous week is accepted.
5. **Create a Codespace** on `week-01` and confirm `echo "$CODESPACES"` prints `true` ([docs/codespaces.md](docs/codespaces.md)).
6. **Store `GROQ_API_KEY` as a Codespaces secret** restricted to this repository. Never commit it.
7. **Create the local environment file** inside the Codespace: `cp .env.example .env`.
8. **Verify prerequisites** (backlog `PREP-07`, `PREP-08`): Groq model access and limits, and the embedding model in a clean Codespace.
9. **Begin `week-01`.** The first engineering task is `W1-01`, which adds the dev container, Compose services, pinned dependencies and the setup, import, test and reset commands. Until then there are no run commands.

## Repository layout

```text
.
├── .github/
│   └── copilot-instructions.md   # development rules for AI assistance
├── docs/                         # requirements, architecture, backlog, process
├── .env.example                  # placeholders only; copy to .env (git-ignored)
├── .gitignore
└── README.md
```

Source code, tests, migrations, the dev container and CI workflows are added during the weekly milestones.

## Branch strategy in brief

```text
main ──► week-01 ──► week-02 ──► week-03 ──► week-04
```

Each week branch starts from the accepted end of the previous one and is reviewed against its predecessor. Accepted milestones get immutable tags such as `cohort-01-week-02-v1`; fixes move forward and publish a new tag version. Week 4 is evaluation, repair and release only. See [docs/branching.md](docs/branching.md).

## Secrets and data

- The GitHub repository is **public**. Do not commit anything you would not publish: secrets, learner data, private source material, instructor keys or the protected holdout evaluation set. Content added to learning packs needs reuse rights that permit public hosting, or must be kept outside the repository. Consider making the repository private before adding pack content, the holdout or learner-related material.
- Never commit `.env`, API keys, AWS credentials, Cognito secrets, database files, generated embeddings or holdout evaluation material. [.gitignore](.gitignore) excludes these.
- `.env.example` contains placeholders only.
- Learning pack content must have documented reuse rights. Text inside a learning pack is treated as data and never as instructions to the application.

## Cost

No resources have been provisioned. The Version 7 planning reserve for the shared pilot is USD 30–35 per month for a 4 GB Lightsail server (an estimate, not a spending cap). Groq inference uses the free tier only. Do not create paid resources without a named owner, budget and cleanup date ([docs/requirements.md](docs/requirements.md), O-07).
