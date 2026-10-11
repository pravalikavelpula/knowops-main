# KnowOps — Demo Script & Proposed Idea

---

## Part 1 — 2-minute demo video script

**Goal of the video:** show that a company can upload its internal documents and every employee can ask questions across that knowledge base — and trust the answers.

**Length:** ~2:00 · **Narration:** ~290 words (normal speaking pace)

### Before you record

- Workspace already created, with **4 ready documents**: `leave-policy.md`, `employee-handbook.md` (says 25 days of leave — conflicts with the policy's 20), `security-runbook.md`, and a Word file `remote-work-policy.docx`.
- Keep **one PDF or DOCX on your desktop** to upload live.
- A second browser window signed in as a **teammate** (Member role).
- A **Finance** group exists; `salary-bands.md` is restricted to it. The teammate is *not* in Finance.
- Light mode, browser zoom 110%, notifications off.

### Script

| Time | On screen | Narration |
|---|---|---|
| **0:00–0:12** | Landing page. Let the hero demo play: question → "Searching 3 documents…" → cited answer. | "Every company has the answers its employees need — buried in PDFs, handbooks and wikis nobody can search. KnowOps turns those documents into a knowledge base anyone can ask, with a source behind every answer." |
| **0:12–0:30** | Sign in → **Documents**. Click **Upload**, drop the file, show it go *Processing → Ready*. | "An admin uploads what the company already has — PDF, Word, Markdown, text or CSV. KnowOps reads each file, splits it into passages, keeps the page or section, and indexes it. A few seconds later it's searchable." |
| **0:30–0:58** | **Ask**: type *"How many days of annual leave do full-time employees get, and can unused days carry over?"* Answer appears. Click citation **[1]** → passage expands with the highlighted line. | "Now any employee just asks in plain language. The answer is written only from the company's documents — and every sentence carries a citation. Click it, and you see the exact passage it came from. No guessing whether the AI made it up." |
| **0:58–1:10** | Point at the answer noting the handbook says 25 days. Ask a follow-up: *"Who approves those requests?"* | "It even spots that the old handbook disagrees with the current policy. And it understands follow-up questions in the same conversation." |
| **1:10–1:28** | Switch to the teammate's window. Ask *"What is the salary range for senior engineers?"* → no salary data, no sources. Back to admin: Access tab on `salary-bands` shows **Restricted · Finance group**. | "Access control carries through to answers. Salary bands are shared only with Finance — so this engineer can't see the document, and can't get its contents out of the assistant either." |
| **1:28–1:45** | **Knowledge health** → **Run scan** → the *20 vs 25 days* conflict appears with both passages side by side. Show the *Unanswered* tab with a logged question. | "KnowOps also keeps the knowledge base honest. A scan flags documents that contradict each other, documents that have gone stale, and questions nobody could answer — so admins know exactly what to fix." |
| **1:45–1:55** | Back on an answer → **Turn into a task** → **Tasks & drafts** → **Approve & create task**. | "And answers turn into work: draft a task or an email from any answer, review it, approve it." |
| **1:55–2:00** | Landing page headline: *Company knowledge you can check.* | "KnowOps — company knowledge you can check." |

### Recording tips

- Ask questions you've rehearsed; the model's wording varies slightly between runs, but the cited facts don't.
- If an answer takes a second, stay on the "Searching your documents…" state — it shows that retrieval is happening.
- Cut between the admin and teammate windows rather than signing in and out on camera.

---

## Part 2 — Proposed idea

KnowOps lets companies upload internal documents (PDF, Word, Markdown, text, CSV) and lets employees ask questions across the whole knowledge base in plain language. Every answer cites the exact passage it came from, and search respects roles, groups and restricted documents, so nothing leaks. It also flags conflicting, stale and missing knowledge, and turns answers into tasks a person approves. Self-hosted, so company data stays private.
