# KnowOps

**Company knowledge you can check.** KnowOps lets a company upload its internal documents and lets employees ask questions across the whole knowledge base. Every answer cites the exact passage it came from, and access rules carry through to search, so nobody gets answers from documents they can't open.


## Features

- **Upload what you already have**: PDF, Word (including tables, text boxes, headers and footers), Markdown, plain text and CSV. Files are split into passages that keep their page or section, then indexed in the background.
- **Answers with citations**: ask in plain language, including follow-ups. Answers come only from your documents, and every claim links to its source passage. If nothing covers the question, the assistant says so instead of guessing.
- **Access control that reaches search**: organization roles (owner, admin, member, viewer), groups, and restricted documents. A restricted document is visible only to its owner, admins and the people or groups it's shared with, in the library *and* in answers.
- **Invite your team**: add people by email. Existing users join immediately; anyone else joins automatically when they sign up with that email.
- **Knowledge health**: questions nobody could answer are logged automatically. An on-demand scan flags documents that contradict each other and documents that have gone stale. Findings can be assigned, resolved or dismissed.
- **Version history**: upload new revisions, compare any two versions line by line, and get a plain-language summary of what changed. The previous version keeps serving answers until the new one is ready.
- **Answers into work**: turn an answer into a task or an email draft. Drafts do nothing until a person approves them, and each answer is drafted at most once.

## How it works

```
Upload ──► extract text ──► split into passages ──► embed ──► Qdrant
                                                                │
Question ──► documents this user may see ──► semantic search ───┘
                                                    │
                            top passages ──► LLM ──► answer + verified citations
```

- **PostgreSQL** is the source of truth: users, organizations, permissions, documents, versions, conversations, findings, tasks.
- **Qdrant** holds passage vectors only. Search is always filtered to the *active* version of documents the user may view, so superseded, archived or restricted text never reaches an answer.
- **MinIO** stores the original files (S3-compatible).
- **Celery + Redis** run document processing and knowledge scans in the background.
- **Citations are checked server-side.** Only sources the model actually cited are stored, renumbered to match.

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 16, React 19, TypeScript, Tailwind CSS 4, Zustand |
| API | Python 3.12, FastAPI, Pydantic, SQLAlchemy 2, Alembic |
| Database | PostgreSQL 16 |
| Search | Qdrant, local embeddings via fastembed (`BAAI/bge-small-en-v1.5`) |
| Background jobs | Celery + Redis |
| File storage | MinIO |
| LLM | Groq: `openai/gpt-oss-120b`, falling back to `openai/gpt-oss-20b` |

## Quick start

### Prerequisites

- Docker and Docker Compose
- For running outside Docker: Python 3.12+ and Node.js 20.9+

### 1. Configure

```bash
cp .env.example .env
```

Then edit `.env`:

| Variable | What to set |
|---|---|
| `SECRET_KEY`, `JWT_SECRET_KEY` | Two different random strings, 32+ characters each |
| `LLM_API_KEY` | Your Groq API key (`gsk_…`) from https://console.groq.com |
| `CORS_ORIGINS` | JSON list of URLs the web app is served from (default: `http://localhost:5173`) |

Everything else has working defaults for local use.

### 2. Run

**Everything in Docker:**

```bash
docker compose up --build
```

The API applies database migrations on start. Open **http://localhost:5173**.

**Or run the apps on your machine** (useful while developing):

```bash
docker compose up -d postgres redis qdrant minio

# Terminal 1 — API
cd backend
pip install -e ".[dev]"
alembic upgrade head
uvicorn app.main:app --reload --port 8000

# Terminal 2 — worker (processes uploads and knowledge scans)
cd backend
celery -A app.services.queue.celery_app worker --loglevel=info

# Terminal 3 — web app
cd frontend
npm install
npm run dev    # http://localhost:5173
```

The first upload downloads the embedding model (~130 MB), so it takes longer than later ones.

### 3. First run

1. **Create an account** at `/register`, then choose **Starting fresh?** to create your organization. You become its owner.
2. **Upload documents** under *Documents*. They show *Processing*, then *Ready*. Only Ready documents are searched.
3. **Ask a question** under *Ask*. Click a citation number to open the passage behind it.
4. **Invite teammates** under *Settings → Members*. Use *Copy the sign-up link* to send them the page; they land in your workspace when they sign up.
5. **Restrict sensitive documents** from a document's *Access* tab, and share them with people or groups (*Settings → Groups*).
6. **Run a scan** under *Knowledge health* to find conflicting and stale documents.

## Configuration reference

All settings live in `.env` (see `.env.example`). The ones you're most likely to tune:

| Variable | Default | Purpose |
|---|---|---|
| `LLM_PRIMARY_MODEL` / `LLM_FALLBACK_MODEL` | `openai/gpt-oss-120b` / `openai/gpt-oss-20b` | Answer models; the fallback is used if the primary fails |
| `LLM_BASE_URL` | Groq | Any OpenAI-compatible endpoint works |
| `EMBEDDING_PROVIDER` | `local` | `local` (fastembed, no key) or `openai` (any OpenAI-compatible embeddings API) |
| `RETRIEVAL_TOP_K` | `6` | Passages given to the model per question |
| `RETRIEVAL_SCORE_THRESHOLD` | `0.2` | Minimum similarity for a passage to be used |
| `CHUNK_SIZE_TOKENS` / `CHUNK_OVERLAP_TOKENS` | `512` / `64` | Passage size when indexing |
| `STALE_DOCUMENT_DAYS` | `180` | Age at which a document is flagged as stale |
| `CONFLICT_SIMILARITY_THRESHOLD` | `0.78` | How similar two passages must be to be checked for a conflict |
| `CONFLICT_MAX_PAIRS` | `12` | Passage pairs sent to the model per scan |
| `MAX_UPLOAD_SIZE_MB` | `50` | Upload size limit |
| `DEBUG` | `true` | Enables `/api/docs`; set `false` when the API is public |

Frontend: `NEXT_PUBLIC_API_URL` (default `http://localhost:8000`) is the API URL as seen from the browser. It is fixed at build time, so rebuild after changing it.

## Roles and access

| | Owner | Admin | Member | Viewer |
|---|:-:|:-:|:-:|:-:|
| Ask questions, read open documents | ✓ | ✓ | ✓ | ✓ |
| Upload documents, create tasks and drafts | ✓ | ✓ | ✓ | |
| See and manage every document | ✓ | ✓ | | |
| Invite and remove people, manage groups, run scans | ✓ | ✓ | | |
| Change roles, grant the owner role | ✓ | | | |

Restricted documents are visible only to their owner, admins and explicit grantees (people or groups). Removing the last grant keeps a document restricted; opening it to everyone is a deliberate action on its *Access* tab.

## Deploying the frontend separately (e.g. Vercel)

The web app can be hosted on its own while the API runs elsewhere:

1. Expose the API over **HTTPS**. For a demo, a Cloudflare quick tunnel works: `cloudflared tunnel --url http://localhost:8000`.
2. On Vercel, set the root directory to `frontend` and the env var `NEXT_PUBLIC_API_URL` to the API URL, then deploy.
3. Add the Vercel URL to `CORS_ORIGINS` in `.env` and restart the API.

Quick-tunnel URLs change on every restart; update `NEXT_PUBLIC_API_URL` and redeploy when that happens.

## Tests

```bash
cd backend
pytest              # needs the postgres container; uses the knowops_test database
ruff check app tests

cd frontend
npm run lint && npx tsc --noEmit && npm run build
```

The backend suite covers authentication, permissions and restricted documents, invitations, uploads and versions, chat and citation handling, knowledge findings, and action drafting. External services (LLM, object storage, search) are replaced with fakes where needed.

## Troubleshooting

| Symptom | Cause / fix |
|---|---|
| Document stuck on *Processing* | The worker isn't running. Start it (see *Run*) |
| Document *Failed: "This file doesn't contain any text"* | The file is blank. Add content and upload again |
| PDF *Failed: "may be a scanned or image-only file"* | No text layer. Export a PDF with selectable text (OCR isn't supported yet) |
| *"The AI engine rejected KnowOps' credentials"* | `LLM_API_KEY` is missing or wrong. Fix it and restart the API and worker |
| Answer has no sources | Nothing in the user's documents covers the question; it's logged under *Knowledge health → Unanswered* |
| Browser shows CORS errors | Add the web app's URL to `CORS_ORIGINS` and restart the API |

Health checks: `GET /health` (process is up) and `GET /ready` (database, Redis, Qdrant and MinIO reachable; 503 otherwise).
API reference (when `DEBUG=true`): http://localhost:8000/api/docs

## Project layout

```
knowops/
├── backend/
│   ├── app/
│   │   ├── modules/          # identity, organizations, documents, ingestion, retrieval,
│   │   │                     # conversations, knowledge_quality, versioning, actions, audit
│   │   ├── services/         # LLM, embeddings, vector store, object storage, queue
│   │   └── shared/           # errors, pagination, helpers
│   ├── alembic/              # database migrations
│   └── tests/                # API and unit tests
├── frontend/
│   └── src/
│       ├── app/              # routes: landing, login, register, /app workspace
│       ├── components/       # screens, UI primitives, marketing pages
│       ├── lib/              # API client, formatting, Markdown renderer
│       └── stores/           # session state
├── docker-compose.yml
└── DEMO.md                   # demo script and project pitch
```

## Known limitations

- Email addresses aren't verified, so anyone who signs up with an invited email joins that organization.
- No OCR: scanned PDFs and images can't be indexed.
- Knowledge scans run on demand, not on a schedule, and assignees aren't notified.

## License

Proprietary — all rights reserved.
