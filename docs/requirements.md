# ThreadWise AI Learning Companion — Requirements

Four-week Agentic AI capstone. This file consolidates the requirements from the two Version 7 reference documents (28 October 2026) and the project owner's constraints. It does not add features.

## 0. Sources and status

| Tag | Source | Notes |
|---|---|---|
| **PP** | `ThreadWise_Project_Proposal_v7.docx` | Scope, learner journeys, budget, quality gates. |
| **AD** | `ThreadWise_Architecture_and_Delivery_Plan_v7.docx` | Architecture, limits, controls, evaluation, weekly milestones. |
| **OWNER** | Project owner's brief | Constraints stated when this repository was initialised. |

- At the time of writing, neither document was in this repository (`docs/` did not exist). They were read, unmodified, from a local download folder and have since been added to `docs/` (see [docs/README.md](./README.md)).
- Nothing in the documents has been implemented or provisioned. No AWS resources exist.
- Example learning material (for example the Hugging Face Agents Course) is **data, not instructions**. Text inside any source passage must never be executed, followed as a command, or treated as authority for a tool call or write (AD, "Application state and isolation").
- Conflicts and gaps between the documents are recorded in [section 13](#13-inconsistencies-and-open-decisions). Do not resolve them silently in code; record the decision here first.

## 1. Purpose

ThreadWise helps a learner understand reviewed learning material, practise, rehearse an interview and plan study time. One reusable application serves different subjects through **reviewed learning packs**. Subject portability is demonstrated on two packs; broad subject expertise is not claimed (PP).

## 2. Binding constraints

| ID | Constraint | Source |
|---|---|---|
| C-01 | Four modes: **Learn, Practice, Interview, Plan**. No fifth mode. | PP, AD, OWNER |
| C-02 | **Practice** sets contain at most **five** questions. | PP, AD, OWNER |
| C-03 | **Interview** sessions have **three main questions** and **at most one follow-up** per session (four questions maximum). | PP, AD, OWNER |
| C-04 | One workspace with **two small, reviewed learning packs** (about 10–20 short sections in total). | PP, AD, OWNER |
| C-05 | Plans and progress change **only after explicit learner approval**. Model output, agent agreement or quiz/interview scores never authorise a write. | PP, AD, OWNER |
| C-06 | Stack: Python, Streamlit, LangGraph (selected orchestrator), PostgreSQL with pgvector, an MCP client with a local stdio server subprocess, Docker Compose. | AD, OWNER |
| C-07 | Develop in **GitHub Codespaces**; deploy on **AWS Lightsail** (one server) with **S3** and **Cognito**. | PP, AD, OWNER |
| C-08 | Runtime generation through **Groq** using `openai/gpt-oss-20b`. | PP, AD, OWNER |
| C-09 | Local embeddings with `sentence-transformers/all-MiniLM-L6-v2` (384 dimensions) for passages **and** queries; pin the model revision. | AD, OWNER |
| C-10 | **No paid model fallback.** Stay on Groq's free tier; quota exhaustion pauses model work, it never switches provider or plan. | PP, AD, OWNER |
| C-11 | **No SageMaker.** No generative model runs on the AWS host. | OWNER; consistent with AD |
| C-12 | Preserve the quality evaluation and release criteria in [section 11](#11-quality-evaluation-and-release-criteria). | OWNER |
| C-13 | Four cumulative weekly branches (`week-01` … `week-04`). Week 4 is evaluation, repair and release; **no new modes or features**. | PP, AD |
| C-14 | Never remove human review, source checks or approval tests to meet a date. | AD |
| C-15 | Keep one planning skill, two agent roles (Planner, Reviewer), one event type and fixed session limits. CrewAI, n8n, alternative vector stores and a public API are optional, rubric-dependent extensions and are **not** in scope unless the assessor requires them. | AD |

## 3. Functional requirements by mode

All modes share authentication, pack selection, the retrieval service, the source panel, the chat components and the Groq adapter. The learner selects a pack and mode, then enters topic, goal and explanation level (beginner or intermediate).

### 3.1 Learn

- **L-01** Explanations are grounded in retrieved passages from the selected pack and cite them.
- **L-02** Wording adapts to the selected level; generated examples are labelled as generated.
- **L-03** Missing goal or time information triggers a clarifying question.
- **L-04** Unsupported requests receive an honest abstention, not an invented answer.

### 3.2 Practice

- **P-01** Generate at most five questions per set, as MCQs or short-answer concept questions, from passages for the selected objective.
- **P-02** Typed schema per item: item ID, type, objective, difficulty, wording, optional choices, expected answer, explanation, evidence IDs.
- **P-03** An MCQ has four distinct choices and exactly one keyed answer. A short-answer item has expected concepts and accepted alternatives.
- **P-04** The Reviewer checks each key against retrieved evidence and flags ambiguity or unsupported content.
- **P-05** Code validates schema, allowed passage IDs, distinct options and question count. One regeneration and re-review of failed items is permitted; if items still fail, show fewer questions with an explanation.
- **P-06** Before submission or explicit reveal, only the learner view is served. Keys and explanations stay on the server and out of client payloads, shared caches and ordinary logs.
- **P-07** After submission, MCQs are compared deterministically; reference answers and formative short-answer feedback cite sources and accept valid paraphrases.
- **P-08** No score marks a lesson complete. Questions are practice material, not predicted or official exam questions.

### 3.3 Interview (text only)

- **I-01** Generate and review three main questions from one topic and level; ask one at a time and wait for the learner.
- **I-02** The Reviewer compares each answer with reference concepts and cited passages.
- **I-03** At most one follow-up in the whole session, chosen from a concrete gap in an answer; it counts toward the four-question maximum.
- **I-04** End with strengths, specific gaps and one suggested learning activity. The summary reflects observed turns and labels limits when answers are missing.
- **I-05** Feedback states what the answer did or did not cover, accepts equivalent valid answers and gives a source-supported improvement.
- **I-06** Blank or insufficient answers receive an invitation to retry or skip, never an invented diagnosis.
- **I-07** Do not infer personality, confidence, employability or examination readiness. No voice, video, live coding or résumé evaluation.
- **I-08** Session cap: 12 model calls and four learner questions, enforced in application state. Sessions can pause and resume.

### 3.4 Plan

- **PL-01** Propose a study plan within the learner's time budget using valid lesson IDs, cited resources, labelled time estimates and prerequisites.
- **PL-02** The proposal is persisted before approval is requested.
- **PL-03** Rejecting, expiring or ignoring a proposal leaves saved plans and progress unchanged.
- **PL-04** Self-reported completion is distinct from assessed mastery. Mastery is never inferred from completion.
- **PL-05** Completion of an approved progress update may create a **pending** next-step suggestion for learners who enabled it; that suggestion needs separate approval. No email, calendar update or automatic plan replacement.

## 4. Data, ingestion and retrieval

- **D-01** Logical records (AD "Minimum data model"): workspace and learning pack; source and passage; plan and progress; proposal and approval; learning session; run and audit event. Gaps are listed in ISS-08.
- **D-02** Packs contain English text, Markdown or text-based PDF, plus a manifest (objectives, order, prerequisites, estimated duration). No OCR, public uploads or open-web crawling.
- **D-03** Preserve source title, file or URL, version, section or page, permitted use and hash. Verify extraction against the original.
- **D-04** Split by headings and the model tokenizer with small overlap; each encoded input stays within the 256 word-piece limit. Preserve paragraph, section and page references before chunking. Cache unchanged vectors by source hash.
- **D-05** `vector(384)` column, cosine search, with **server-enforced** workspace, authorised-pack and version filters.
- **D-06** Start with exact vector search. The required retrieval improvement is hybrid PostgreSQL keyword plus vector search with rank fusion, compared on the same development cases, corpus and model. Reranking is an extension only.
- **D-07** Publish a new pack version atomically; exclude superseded or deleted passages; test one content update. A second pack must load through configuration, not code.
- **D-08** Instructor assessment keys and holdout references are kept outside retrieval.
- **D-09** An operator imports reviewed packs with a command and manifest. No authoring portal.

## 5. Agents, MCP, skill and memory

- **A-01** LangGraph coordinates two roles with separate prompts, message state and tool allowlists. The Planner decomposes the goal, reads evidence and drafts a plan or questions. The Reviewer checks output against the mode rubric (accept or revise for plans and questions; formative feedback for learner answers).
- **A-02** One revision and one final review, then stop with an explanation. Neither agent can approve a write.
- **A-03** Per run: at most **eight model calls** and **four tool rounds**, including both roles, retries and the revision. One active inference run per deployment; release the slot while waiting for learner input.
- **A-04** Exceeding a cap pauses or ends generation without changing plans or progress, preserves a resumable checkpoint and explains the limit. Rate limiting uses bounded backoff with a visible retry message.
- **A-05** Use a real MCP client and a local stdio server subprocess. Trusted learner and pack context is bound **outside** model arguments; enrolment is enforced in every tool; the database role is read-only; the subprocess is closed after each run. No public MCP endpoint. The MCP server cannot commit plan or progress changes.
- **A-06** Tools named in AD: `search_lessons`, `read_lesson`, `get_progress`, `propose_study_plan`, `propose_progress_update`, `apply_approved_change` (see ISS-03 for the MCP boundary).
- **A-07** One versioned study-planning skill (`SKILL.md`, instructions, plan schema, reviewed examples). Record its version in each run; show reuse on both packs. Tool permissions are enforced in code regardless of skill text.
- **A-08** A mode selector routes Learn, Practice and Interview to bounded prompt templates using the same read tools. Session transitions and question limits are controlled by application code, not by the model.
- **M-01** Memory: short-term (conversation and task state); long-term (learner-approved goals, plans, progress); episodic (previous proposals and learner-confirmed feedback by owner and pack). Retrieve only relevant, bounded episodes.
- **M-02** Raw practice and interview scores stay session observations; only explicitly saved preferences or progress enter long-term memory. Persist checkpoints for restart. Never store private model reasoning.

## 6. Approval and write path

1. Persist the proposal (exact changes, evidence, expected version, expiry, owner, reviewer).
2. On approval, validate the authenticated owner, proposal ID, expiry and expected record version.
3. Apply the change, its audit event and (for progress) an outbox event in **one database transaction** with an idempotency key.
4. A rejected, stale, expired, repeated or wrong-owner request must cause no write.

Streamlit session state is a UI convenience only. Reruns and repeated clicks must not repeat actions.

## 7. Event automation

- **E-01** Outbox pattern on the same Lightsail server: a worker claims pending events with a lease, bounded retries and states pending / processing / completed / failed.
- **E-02** Delivery is at-least-once. A unique event-to-proposal key prevents duplicate effects.
- **E-03** Recheck enrolment and record versions before processing and before applying.
- **E-04** A schema change or quota failure leaves the event recoverable. Test a crash after proposal creation and before acknowledgement.

## 8. Security, privacy and isolation

- **S-01** Cognito sign-in through Streamlit OIDC; pre-created pilot users; public sign-up disabled. Authentication identifies a user; pack authorisation is a separate application allowlist.
- **S-02** Enforce workspace, pack and owner filters in every tool and citation fetch. Model-suggested pack IDs grant nothing.
- **S-03** Never cache learner-specific results in a shared cache without an owner-specific key.
- **S-04** Secrets live in Codespaces secrets and restricted server configuration, never in the repository. Exclude secrets and private reasoning from logs; keep logs bounded.
- **S-05** The database and development ports are not publicly exposed. Only HTTPS and its HTTP redirect are public; SSH is limited to the operator.
- **S-06** Store owner-scoped session checkpoints, turns and feedback under a visible session-history notice. Session observations do not alter approved goals, plans or completion records.
- **S-07** Pin each session to a pack version and stop for review if that version is withdrawn.
- **S-08** Only permitted excerpts may be sent to Groq (see ISS-04 for the payload definition still to be agreed).
- **S-09** Do not expose Groq keys to untrusted pull requests.

## 9. Model and resource limits

| Limit | Value | Source |
|---|---|---|
| Model calls per run | 8 | AD |
| Tool rounds per run | 4 | AD |
| Model calls per interview session | 12 | AD |
| Learner questions per interview | 4 (3 + 1 follow-up) | PP, AD |
| Questions per practice set | 5 | PP, AD |
| Active inference runs per deployment | 1 | AD |
| Keys | Personal key per developer; one operator-managed key for the shared pilot; never multiply keys to bypass quota | AD |
| Pilot accounts | Up to about five | PP, AD |

Tune limits only after measurement. Record latency, tokens, calls and quota failures per request, practice set and complete interview, plus cold and warm startup memory.

## 10. Deployment, operations and cost

- **O-01** Same container definitions in Codespaces and on Lightsail. Compose runs app, database and worker; a production override adds the HTTPS proxy (WebSocket support). The encoder is packaged in the image or a persistent cache.
- **O-02** Lightsail: start sizing around 4 GB RAM (hypothesis until measured together). Persistent host volume for PostgreSQL; private container network.
- **O-03** S3: private bucket, encryption, versioning, defined retention; holds source originals and database exports. No public links for private material.
- **O-04** Daily PostgreSQL export during the pilot and before each migration; test a **full restore**. The server volume is not a backup.
- **O-05** Release procedure: build a pinned image from an accepted tag, back up, migrate, run production Compose, keep the previous image and a compatible backup.
- **O-06** Operator patches the server, rotates secrets and reviews bounded logs. Test restart, rejected approvals, quota exhaustion and a restored learner plan.
- **O-07** Budget (USD per month, checked 26 October 2026, before tax): Lightsail 4 GB 24; S3 1–2; optional snapshots 1–3; Cognito 0 expected; Groq and embeddings 0. Estimate 25–26 (26–29 with snapshots). **Reserve 30–35**; reserve 50–55 if the 8 GB server (44) is needed. Alerts are not a spending cap. Stopped Lightsail servers still bill. Record account owner, budget target and cleanup date; delete unneeded billable resources after backing up.
- **O-08** Excluded from the estimate: labour, taxes, new domains, GitHub overages, messaging, excess transfer, Copilot subscription.
- **O-09** No high-availability guarantee. RDS, autoscaling and multi-workspace hosting are later phases.

## 11. Quality evaluation and release criteria

Preserved from Version 7. Thresholds are agreed before final testing.

### 11.1 Dataset

36 human-reviewed scenarios: 24 development, 12 protected holdout. Each mode appears in each pack. Preserve denominators.

| Mode | Development | Holdout | Holdout coverage |
|---|---|---|---|
| Learn | 8 requests | 4 requests | 3 answerable, 1 unsupported; development adds level changes and conflicting passages |
| Practice | 6 sets | 3 sets of 5 (15 items) | MCQ and short answer mix; both packs; alternative answers; ambiguous distractors |
| Interview | 6 sessions | 3 sessions | 3 main questions + 1 follow-up each = 12 feedback turns + 3 summaries; correct, partial, incorrect and blank answers; valid alternatives |
| Plan | 4 requests | 2 requests | Different time budgets, prerequisites and progress; both packs |

Each case records pack and source hashes, objective, expected behaviour, reference concepts, supporting passage IDs and rubric. Development also covers insufficient evidence, prompt injection and quota interruption. Authored learner answers are labelled fixtures, not real user data.

### 11.2 Gates

| Gate | Threshold |
|---|---|
| Learn and retrieval | All four holdout requests pass correctness, relevance, level and uncertainty checks. Mean Recall@5 ≥ 0.80 on answerable cases with labelled passages. |
| Citation support | ≥ 90% of audited factual claims supported across all modes (report counts by mode). Any material unsupported claim fails regardless of the aggregate. |
| Practice | All 15 keys and explanations factually correct and supported; ≥ 14 of 15 items pass the rubric. |
| Interview | All three sessions pass the feedback rubric on every turn and summary. No invented learner error or unsupported readiness claim. |
| Plans and controls | Both plans satisfy valid IDs, time and prerequisites. All ten action-control tests and all mode, MCP, memory, event and CI checks pass. |

**Ten action-control tests:** approve, reject, expired, stale, duplicate, restart, wrong owner, embedded instructions, wrong pack, revoked access. Test actual state and audit events.

**Mode checks (in addition):** question limits, one keyed MCQ answer, hidden keys, one follow-up maximum, pause/resume, blank answers, conflicting evidence, pack withdrawal.

### 11.3 Rubrics

- **Practice item hard checks:** answerable from the selected evidence; correct key; supported explanation; exactly one defensible MCQ answer. A wrong key, unsupported explanation or ambiguous correct option is a release failure.
- **Practice dimensions (0/1/2):** clarity; objective and level; assessment value. Pass = every dimension ≥ 1 and total ≥ 5 of 6 after hard checks.
- **Interview feedback dimensions (0/1/2):** accuracy and grounding; actionable guidance; fairness and tone. Every turn and summary needs every dimension ≥ 1 and total ≥ 5 of 6. Any fabricated learner error or material unsupported correction fails the session.

### 11.4 Independent assessment

- Humans review all holdout outputs: 15 practice items, 12 interview feedback turns, three summaries, and the Learn and Plan outputs.
- The release evaluator is a separate model call from the application Reviewer, using the written rubric, sources and reference concepts.
- Audit at least 12 judge ratings against human labels (four Learn, four practice, four interview); include failures and valid alternatives from development outputs. Record pass/fail agreement, per-dimension differences, false passes and false failures; a second reviewer adjudicates where available.
- Human decisions govern; neither a model vote nor a mean score overrides a hard failure. Same-model generation and judging can share errors.
- Seed wrong keys, ambiguous MCQs, a correct alternative answer, unsupported criticism and overconfident summaries during development.

### 11.5 Freeze and decision

- Freeze code tag, corpus hash, model ID, prompts, skill and rubrics before the holdout. Run the holdout **once**. A changed build needs a new versioned evaluation; do not tune on holdout answers or present repeated attempts as independent.
- **RELEASE:** every agreed quality gate, deterministic control and deployment check passes with complete live evidence.
- **HOLD:** any failed threshold or material correctness, feedback, authorisation or duplicate-write defect.
- **INCONCLUSIVE:** quota failures or missing evidence prevent a decision **and** no known blocking failure already requires HOLD. A missing live run is INCONCLUSIVE, not a pass.
- The pilot cannot establish universal accuracy or improved learning outcomes.

### 11.6 Continuous integration

GitHub Actions on pull requests and pushes to reviewed branches runs schema, retrieval-fixture, practice-key visibility, interview state, authorisation, approval and event tests. Replayed model outputs are labelled fixtures (they verify plumbing, not live quality). A deliberately failing control must block the check. The test report is a small, short-lived CI artifact. Live Groq evaluation is manually triggered on trusted code with protected secrets.

## 12. Delivery constraints

- Cumulative branches `week-01` to `week-04`, reviewed against the predecessor, with immutable tags such as `cohort-01-week-02-v1`. `main` is the stable guide; deploy only a reviewed release. Propagate fixes forward as new tag versions; never rewrite learner baselines.
- If week 3 is incomplete, reduce topic breadth and visual polish first; any other deferral needs an explicit scope decision.
- Deliverables: branches and tags, working application, source manifest, evaluation dataset and report, MCP trace, skill package, CI report, curriculum checklist, deployment/operating guide, short demonstration.
- Curriculum M0–M6 coverage is a planned evidence checklist; the complete rubric has **not** been checked. Each topic is marked implemented, conceptual, deferred or awaiting rubric confirmation. LangGraph and the worker do not prove use of CrewAI, CrewAI Flows or n8n; pgvector does not demonstrate FAISS or Qdrant.

**Deferred (out of scope):** shared multi-workspace hosting, LMS integration, formal grading, full mock exams, voice/video, code execution, open-web discovery, OCR, public uploads, authoring portal, email/calendar, scheduled automation, A2A, GraphRAG, public `/ask` API, reranking.

## 13. Inconsistencies and open decisions

Each item must be decided and recorded here before the dependent work starts. "Blocks" names the first backlog item affected.

| ID | Issue | Evidence | Proposed resolution | Blocks |
|---|---|---|---|---|
| ISS-01 | **Week 2 Practice needs the Reviewer; agents land in week 3.** Practice requires Reviewer key checks and the planning skill, but the Planner/Reviewer handoff is listed under week-03 (AD milestones). | AD week-02 vs week-03; PP week table | Build a minimal Planner→Reviewer path with bounded calls in week-02 for Practice only; extend to Interview, Plan and events in week-03. Confirm. | W2-06, W2-08 |
| ISS-02 | **Deployment completion timing.** PP week 4 says "complete deployment"; AD says complete the Lightsail pilot by end of week 3 and use week 4 for fixes. Cognito is validated in week 1 but "authentication" is a week-03 deliverable. | PP weeks table; AD delivery safeguards | Week 1 spike, week 3 authenticated pilot, week 4 release deploy and fixes only. Define "complete" explicitly. | W1-09, W3-13 |
| ISS-03 | **MCP boundary.** The tool list includes `propose_*` and `apply_approved_change`, yet the MCP server must be read-only and cannot commit changes; persisting a proposal is itself a write. | AD tool sequence vs MCP section | MCP exposes only read tools (`search_lessons`, `read_lesson`, `get_progress`). Proposal persistence and `apply_approved_change` are application-side with their own write role. Proposals are not plans. | W2-04, W3-06 |
| ISS-04 | **Outbound data to Groq is under-defined.** AD says "permitted question and source excerpts" leave AWS, but interview answers, goals, level and saved memory are also needed. | AD overview vs interview and memory sections | Define a per-mode payload allowlist, minimisation rules and exclusions; confirm provider data handling and learner notice. | PREP-09, W1-05 |
| ISS-05 | **Citation gate scope.** PP lists the 90% claim-support gate under Learn; AD applies it across all modes. | PP quality table; AD gates | Treat as an all-mode gate (as in section 11.2). | W4-01 |
| ISS-06 | **INCONCLUSIVE precedence.** PP says missing live evidence requires INCONCLUSIVE; AD adds "and no known blocking failure already requires HOLD". | PP evidence section; AD decision rules | Use the AD wording (HOLD takes precedence). | W4-01 |
| ISS-07 | **Holdout protection vs version control.** Fixtures and expected results live in version control and learners work in copies containing all weekly branches, but holdout references must stay protected. | AD CI and delivery workflow vs holdout protection | Keep holdout scenarios, references and results in a separate access-controlled location, released to the evaluator only at freeze. Keep only development fixtures in learner repos. | PREP-05 |
| ISS-08 | **Data model gaps.** The minimum data model omits enrolment/allowlist, next-step-suggestion preference, outbox events and leases, workflow checkpoints, episodic memory, session-history notice, skill version and model-call counters, all of which other sections require. | AD data model vs sections on events, memory, limits | Add these as explicit logical entities in `architecture.md` and migrations. | W1-02 |
| ISS-09 | **Daily backups vs no runtime S3 credential.** Daily exports are required, but uploads use short-lived operator credentials and the runtime needs no S3 key. | AD access procedure vs recovery | Decide who or what runs daily exports (operator job, or a narrowly scoped write-only identity), failure detection, retention, RPO and RTO. | W3-14 |
| ISS-10 | **Role of S3 originals.** Citations may reference a "source URL or S3 key", but the runtime has no S3 access and private originals have no public links. | AD data model vs access procedure | Serve citations from stored passages (with source metadata); originals are operator-only. Confirm. | W2-02 |
| ISS-11 | **Call and quota budget unmeasured.** An interview needs question generation, review, up to four feedback turns and a summary within 12 calls, while one inference slot is shared with the event worker and evaluation runs. Groq limits are account-wide and include token limits. | AD model access section | Measure in weeks 1–3; define slot priority and queue behaviour for learner runs vs worker vs evaluation; persist call counts across pause/resume. | W2-06 |
| ISS-12 | **Plan quality coverage.** The plan gate checks IDs, time and prerequisites only, although plans must also have cited resources and labelled estimates; the judge audit has no Plan sample. | AD tool sequence vs gates and audit | Add deterministic checks for citations and labels; confirm human-only review is acceptable for Plan quality. | W3-06 |
| ISS-13 | **Practice fallback vs fixed denominator.** Fewer than five questions may be shown at runtime, but the holdout requires 15 items. | AD practice workflow vs gates | Evaluation counts a short set as a failure for missing items, or the holdout is rerun under a new version; decide before freeze. | W4-01 |
| ISS-14 | **Who builds.** Documents say learners implement while the developer prepares scaffolding; this project may be built by a single team. Weekly hours and group size are unset. | PP delivery scope; AD preparation | Name roles and effort in PREP-02. | PREP-02 |
| ISS-15 | **Unspecified versions.** Python, Streamlit, LangGraph, MCP SDK and Postgres/pgvector versions are not given. | AD (pinned dependencies only) | Pin in week 1 and record. | W1-01 |
| ISS-16 | **Rubric and named tools.** The complete capstone rubric was not checked; graded use of CrewAI, n8n, FAISS, Qdrant or Langfuse is undecided. | PP/AD curriculum sections | Obtain assessor decision before week 1. Until then, none is built. | PREP-01 |
| ISS-17 | **Codespaces deferred.** C-07 requires development in GitHub Codespaces, but the owner decided on 30 October 2026 to proceed without one for now. Whether the organisation allows or funds Codespaces for members was not confirmed. The only local machine is Windows with Python 3.11 and no Docker, so Compose and pgvector cannot run locally. | Owner decision; `docs/codespaces.md` | C-07 stays binding. Until a Codespace (or local Docker) exists, work is limited to authoring and documentation. Anything needing a running stack (clean-startup, encoder memory, dev container validation) is not accepted until verified in a Codespace. Record the billing owner and result when one is created. | W1-01, PREP-08, W1-12 |

## 14. Preparation requirements

Entry criteria completed **before** week-01 starts (PP, AD "Readiness"). Each has an owner and a completion record in `docs/backlog.md`.

1. Confirm workspace owner, content owner, content rights and external Groq processing (PREP-03, PREP-09).
2. Source the first pack (public starter candidate: Hugging Face Agents Course Unit 1; verify the licence of the specific assets used and preserve notices) and a **second, authentic pack from a different subject** (PREP-03, PREP-04).
3. Reviewer drafts ten pilot questions, two study plans, one five-question practice set and one short interview with reference answers (PREP-03).
4. Reviewer labels the 36 evaluation scenarios and protects the holdout (PREP-05).
5. Prepare starter adapters for MCP, Cognito/OIDC, outbox/events and CI (PREP-06).
6. Verify Groq account access, model availability and actual limits; store the key securely; smoke-test tool and structured output (PREP-07).
7. Run the MiniLM encoder in a clean Codespace; confirm chunk lengths, vector size and memory (PREP-08).
8. Confirm pilot users, AWS region, hostname, budget owner, Cognito configuration and Codespaces entitlement (PREP-10).
9. Agree weekly hours, team size, roles and quality thresholds (PREP-02, PREP-11).
10. Obtain the assessor's curriculum and named-tool decisions (PREP-01).

If prepared components are unavailable, extend the timeline or agree a reduced scope; do not compress evaluation into the final day.
