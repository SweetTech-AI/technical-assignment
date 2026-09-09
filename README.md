# SweetTech — Take-Home Assignment

**Level:** Senior Full-Stack Engineer
**Time budget:** 6 hours — a hard constraint, not a suggestion
**Deadline:** 5 days from receipt

---

## The story

You are a senior engineer at SweetTech, a fintech building modern tooling for
agricultural lenders. Six people on the team, no design department, more ambition
than runway.

This morning you sat in on a two-hour meeting with PrairieVault Financial — a
mid-sized ag lender managing loan portfolios across four states. Their current
system is a spreadsheet shared over email. They are ready to move.

The conversation was good. Their head of lending, Cathleen, walked you through
how her team works: loan officers log in every morning, scan a list of active
borrowers, check which documents came in overnight, and decide who to act on
next. She showed you what a typical borrower file looks like. She mentioned they
have been burned before by vendors who overpromised. She wants to see something
real before she signs anything.

Toward the end of the meeting Cathleen leaned forward and said something you
wrote down verbatim:

> "Not everyone on my team should see everything. A junior officer handles the
> intake. The sensitive stuff — the financial details, the risk data — that stays
> with the senior team."

Your CTO leaned over on the way out of the building and said: *"We demo next
Friday. Make it look like we have been building this for a year."*

You have one week. You have your editor, your terminal, and your AI tools. Build
the demo. Make it count. And write it well enough that it becomes the foundation
of the real feature — because if Friday goes well, Cathleen is going to want it
in production by Q2.

---

## What you are building

A loan officer dashboard for PrairieVault's internal team. This is not a
customer-facing interface. Cathleen's team will use this every morning.

The centerpiece is a borrower management table: a paginated, searchable, sortable
list of borrowers with their application status, submitted documents, and
available actions. It should feel fast, clear, and professional. Loan officers
are busy. Every extra click costs them.

---

## Tech stack

| Layer    | Stack                                   |
| -------- | --------------------------------------- |
| Backend  | FastAPI, Pydantic, SQLAlchemy, SQLite   |
| Frontend | Angular (latest stable)                 |
| Run      | One command, documented in your README  |

SweetTech's production backend is Python, which is why this assignment is
Python. If you have not shipped a lot of FastAPI, that is fine and expected — we
are reading for how you design an API and where you put a trust boundary, not for
framework trivia.

**Use SQLAlchemy for all database access** — models, relationships, and queries.
Raw SQL is not acceptable. For schema setup, a startup script that creates tables
if they do not exist is fine. If you prefer Alembic, use it — but do not let it
eat your week. Document your choice in `DECISIONS.md`.

**Running it:** a new engineer must get from cold clone to running app in under
five minutes, following only your README. `docker-compose up` is welcome if you
already have that muscle memory, but it is not required and it is not scored — a
`make dev` or a documented `uvicorn` + `ng serve` pair is equally acceptable.
Spend the time you save on the API and the tests.

---

## The domain

PrairieVault manages agricultural borrowers — farms and operators applying for
credit lines. From what Cathleen described, every borrower sits in one of these
pipeline states:

```
applied → in_progress → review → approved
                              └→ declined
                                        → closed
```

Each borrower submits documents as part of their application. Cathleen mentioned
land titles, tax returns, crop insurance certificates, income statements, bank
statements, and a few others. Each document is reviewed individually and marked
accordingly.

Cathleen did not give you a formal spec. This is what you remember from the
meeting and the notes you typed on the way back to the office. Some details are
yours to infer. Make reasonable decisions and write them down.

---

## The dashboard table

The main view is a borrower table. Loan officers need to move fast, so the table
must support:

- Search by borrower name or farm name
- Sort by a single column, ascending and descending
- Filter by borrower status
- Pagination with a configurable page size
- Status badges that are visually distinct at a glance
- The three row actions described below

**All filtering, searching, sorting, and pagination happens on the server.** The
frontend sends parameters. The backend does the work.

---

## Row actions

Each row has exactly three action buttons. They are **always visible**, for every
borrower status and every user role:

| Button             | Opens                              |
| ------------------ | ---------------------------------- |
| **View Profile**   | Borrower details                   |
| **View Documents** | The submitted document list        |
| **Credit Line**    | The amount and terms requested     |

There is no server-driven action logic and no conditional rendering based on
status. The buttons are always there. What the user *sees* when they click is
where the role-based visibility rules apply — that is the interesting part, and
it belongs in the API, not in the button list.

Whether these open a side panel, a modal, or separate routes is your call.
Document it in `DECISIONS.md`.

**View Profile** shows the borrower's information and their submitted document
list. Each document shows at minimum its type, upload date, and status (pending,
verified, rejected).

---

## Roles and data visibility

PrairieVault has two types of internal users:

| Role                    | Responsibility                                          |
| ----------------------- | ------------------------------------------------------- |
| **Loan Officer**        | Intake and day-to-day borrower communication            |
| **Senior Loan Officer** | Underwriting, risk assessment, and approvals            |

Cathleen was clear that not all data should be visible to everyone. Some fields
and document details are sensitive enough that only Senior Loan Officers should
see them.

**You decide which fields are restricted.** Think about what makes sense in a
real lending operation — what a junior officer needs versus what should stay with
the senior team. Own the decision and document it in `DECISIONS.md`.

The restriction must be **enforced by the API**. The server inspects the caller's
role and shapes the response accordingly, omitting or masking restricted fields
before the data ever leaves the backend. The frontend renders what it receives.
It does not decide what to show based on role. It does not receive data and hide
it. If a field is not meant for a Loan Officer, that field is not in the
response.

You do not need a full authentication system. A simple mechanism for switching
between the two roles is enough — something you can use during the demo to show
Cathleen that the data changes depending on who is logged in. Keep it honest: the
enforcement must be real, not theatre.

---

## Seed data

The demo needs to look real. Seed the database automatically on first start with
exactly **100 borrowers**.

**Five anchor records** — one for each active status (applied, in_progress,
review, approved, declined). Give these real-feeling names, realistic farm
operations, loan amounts that make sense, and document sets that match their
stage. A borrower who just applied would have two or three docs. One sitting in
review should have a nearly complete file.

**The other 95** are generated programmatically. Mix up farm types: grain,
livestock, dairy, mixed operations, specialty crop. Loan amounts should feel like
real ag lending — $50,000 on the low end, up to $2,000,000 for the big operators.
Document sets grow with pipeline stage.

Seeding must be **deterministic**: same data every run.

Also seed **at least one Loan Officer and one Senior Loan Officer** so the
role-based visibility can be demonstrated out of the box.

Cathleen is going to be sitting across the table watching you scroll through this
list. It needs to feel like her world.

---

## Documents

Documents already exist in the system. **There is no file uploader.** Assume files
were uploaded through a separate borrower-facing process and are stored in a
cloud object store. The database holds the metadata and a storage URL. The
dashboard displays that metadata and links to the file. It does not upload,
replace, or delete anything.

Seed document records with realistic metadata and plausible storage URLs. The
files do not need to exist —
`s3://prairievault-docs/borrowers/{id}/{filename}` or similar is enough.

Document links do not need to download anything real. Every link can point to the
same sample PDF, open a blank tab, or simply look like a link. What matters is
that the metadata is there and the UI makes it clear a file is accessible. We are
not testing file serving.

Some document metadata may be sensitive. Apply the same visibility rules here as
you do for borrower fields — the API decides what each role sees.

---

## API design

Build a REST API. **Design the contract yourself** — Cathleen did not hand you a
spec, she described a workflow. There is intentional ambiguity here and that is
the point. We want to see the choices you make and why.

This is a read-heavy dashboard. You do not need full CRUD. The one write
operation required is a `PATCH` to update a borrower's status, and only to
`approved` or `declined`. That is the one decision a loan officer makes. No other
transitions are required. Everything else is `GET`.

Constraints:

- Pagination via limit/offset or cursor — your call, own it
- Single-column sort as a query parameter
- A consistent response envelope across all list endpoints
- Proper HTTP semantics and meaningful error responses
- All inputs validated with Pydantic
- Role-based field visibility enforced at the response serialization layer
- `PATCH /borrowers/{id}/status` is the only write endpoint required

---

## Testing

This is going into production if Friday goes well. Write tests like it.

**Backend — pytest.** Cover at minimum:

- Pagination edge cases: last page, empty result, out-of-range offset
- Search returning no matches
- Sort parameter validation: bad column name, bad direction
- Status filter combinations
- `PATCH` status: valid transitions, and rejection of anything other than
  `approved` / `declined`
- A Loan Officer does not receive restricted fields
- A Senior Loan Officer receives the full response
- **Restricted fields are absent from the response, not merely null**

**Frontend — your call.** You have a demo in a week and a backend to stabilize.
If you write frontend tests, great, we will read them. If you decide the time is
better spent elsewhere, say so in `DECISIONS.md`. Cutting scope deliberately and
honestly is a skill. Cutting it silently is not.

We are not counting lines. We are looking for tests that would catch a real
regression before it reaches Cathleen — and for engineers who know when writing
more tests is the right move and when it is not.

---

## What we are *not* scoring

So you can spend your six hours where they count:

- Visual design polish beyond clean and legible
- Containerization, if a documented local run is simpler for you
- Frontend test coverage, if you tell us why you skipped it
- Authentication, beyond the role switch
- Any endpoint not listed above

---

## `DECISIONS.md`

Short file, repo root. Three to five paragraphs covering:

- Which fields you restricted and why
- Your pagination choice and why
- Your document fetch strategy and why
- How you surfaced role differences in the UI without the frontend making the
  decision
- Anything the brief left ambiguous and how you resolved it
- What you would do differently with more time or a proper spec

We read this to understand how you think under ambiguity, not to grade your
writing.

---

## `AGENTS.md` (optional)

If you used an AI coding assistant — Cursor, Copilot, Claude, anything — we would
love to see the context file you gave it.

This is not a requirement. No penalty for skipping it, no bonus points for
padding it out. We are simply curious: what did you tell it about the project?
What constraints did you set? What did you tell it *not* to do?

A genuine `AGENTS.md` written for your own workflow tells us more than a polished
one written for us. If it exists, drop it at the repo root and we will read it as
a window into how you work, not as a deliverable to grade.

---

## Submission

Create a private Git repository and grant read access to the account provided.
Everything needed to run the project must be in the repo. No manual steps beyond
what is in your README.

---

## Evaluation

We are looking for the person who could have walked out of that meeting with
Cathleen and built something worth showing — in six hours.

| Criterion            | What we mean                                                        |
| -------------------- | ------------------------------------------------------------------- |
| **API design**       | Consistent, correct, validated, role-aware at the right layer        |
| **Role enforcement** | The boundary is real and the tests prove it                          |
| **Code quality**     | Clean, intentional, no dead code, no shortcuts that bite later       |
| **Testing**          | Catches real regressions, including the visibility rules             |
| **Detail and craft** | Loading states, error states, empty states, edge cases               |
| **Scope decisions**  | What you cut, why you cut it, and that you were honest about it      |
| **Decisions**        | Clear thinking under ambiguity and time pressure                     |

The gaps in this brief are deliberate. A senior engineer at a startup notices
them, makes a call, keeps moving, and writes it down. Shipping something focused
and honest beats shipping something sprawling and half-finished. That judgement
is part of what we are hiring for.

Good luck. We are rooting for you — Cathleen is too, she just does not know it
yet.
