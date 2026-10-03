## 1. Core principle

The system must NOT make an authoritative legal, regulatory, or funding-eligibility decision.

It should only assess the supplied grant guideline and supplied application/evidence and clearly communicate:

- satisfied requirements
- missing requirements
- weak evidence
- ambiguous evidence
- unsupported claims
- missing supporting documents
- clarification questions
- reviewer decisions

Every AI-generated mapping must contain a source citation pointing to the relevant supplied guideline/application evidence.

Do not use unrestricted external grant databases or external legal research.

---

# 2. Recommended technology stack

Use:

### Backend

- Python 3.12+
- FastAPI
- SQLAlchemy
- SQLite for local development
- PostgreSQL-compatible design
- Pydantic
- pytest
- Uvicorn

### Frontend

- React
- TypeScript
- Vite
- React Router
- modern responsive CSS
- accessible UI components

### AI layer

Create a provider abstraction so the application works in two modes:

1. Demo/mock mode with deterministic sample analysis
2. Real LLM mode through an environment-configured API

Never hard-code API keys.

Example:

```env
AI_PROVIDER=mock
AI_API_KEY=
AI_MODEL=

```

The mock mode must work without an external API key.

---

# 3. Main user workflow

The user should be able to:

1. Create an assessment
2. Upload/provide a grant guideline
3. Upload/provide a draft application
4. Optionally add supporting-document metadata
5. Run analysis
6. View extracted requirements
7. View application-to-requirement mappings
8. Inspect evidence and citations
9. Review/correct/reject mappings
10. Track missing supporting documents
11. Review unsupported claims
12. Review clarification questions
13. See deterministic completeness metrics
14. Generate a reviewed completeness summary/report
15. See version history and audit history
16. Be warned when an assessment becomes stale

---

# 4. Input model

Support:

### Grant guideline

- title
- document name
- version
- content
- upload date
- content hash

### Draft application

- title
- applicant
- version
- content
- upload date
- content hash

### Supporting documents

Each document should have:

- name
- type
- description
- status:
  - provided
  - missing
  - pending
- optional reference/location
- version

For the MVP, text input plus TXT/DOCX/PDF upload is sufficient.

OCR is NOT required.

If an uploaded document cannot be parsed, return a clear user-facing error instead of silently failing.

---

# 5. Requirement extraction

Extract requirements from the supplied guideline.

Each requirement should have:

```json
{
  "id": "REQ-001",
  "text": "Applicants must submit an itemized project budget.",
  "category": "submission",
  "priority": "mandatory",
  "source_citation": {
    "document": "Grant Guideline",
    "section": "3.2",
    "quote": "..."
  }
}

```

Requirement priority must distinguish:

- mandatory
- recommended

Do not treat recommendations as mandatory requirements.

Categories can include:

- eligibility
- submission
- documentation
- budget
- timeline
- impact
- governance
- other

---

# 6. Application mapping

Map application evidence to every extracted requirement.

Each mapping should contain:

```json
{
  "requirement_id": "REQ-001",
  "status": "satisfied",
  "confidence": 0.91,
  "evidence": "The application includes an itemized project budget.",
  "application_citation": {
    "section": "Budget",
    "quote": "..."
  },
  "reason": "The supplied application contains the required budget information."
}

```

Allowed AI statuses:

- satisfied
- missing
- weak
- ambiguous

Every mapping MUST include evidence/citation when evidence exists.

If there is no evidence, explicitly say so.

---

# 7. Human review

Every mapping must be reviewable.

Allow the reviewer to:

### Confirm

Accept the AI mapping.

### Correct

Change the status/evidence/citation and provide a note.

### Reject

Reject the mapping and require a reason.

Example review payload:

```json
{
  "decision": "correct",
  "corrected_status": "satisfied",
  "corrected_evidence": "...",
  "note": "Human reviewer verified the evidence."
}

```

Maintain the original AI result.

Never overwrite the original AI assessment.

Store reviewer decisions as separate records.

---

# 8. Deterministic completeness calculation

The completeness score MUST be calculated by backend code, not by the LLM.

For mandatory requirements:

```text
completion % =
satisfied mandatory requirements
/
total mandatory requirements
× 100

```

Only reviewer-confirmed/corrected mappings should count as reviewed complete items when applicable.

Expose deterministic counts:

- mandatory total
- satisfied
- weak
- missing
- ambiguous
- rejected
- pending review
- completion percentage

Also calculate recommended requirement counts separately.

Never allow the AI to directly decide the final percentage.

---

# 9. Supporting document tracking

Track required supporting documents.

Example:

| Document              | Required    | Status   |
| --------------------- | ----------- | -------- |
| Project Budget        | Yes         | Provided |
| Board Approval Letter | Yes         | Missing  |
| Sustainability Plan   | Recommended | Missing  |

Expose:

- total required documents
- provided
- missing
- pending

Allow the reviewer to update document status.

---

# 10. Unsupported claims

Identify application claims that are not supported by supplied evidence.

Example:

> "Our project will reduce unemployment by 40%."

If the supplied evidence does not support this claim, classify it as:

```text
unsupported

```

Each unsupported claim should contain:

- claim
- application citation
- reason
- related requirement if applicable
- reviewer status

Do not invent external evidence.

---

# 11. Clarification questions

Generate useful clarification questions based only on:

- missing requirements
- weak evidence
- ambiguous mappings
- missing supporting documents
- unsupported claims

Example:

```text
Please provide evidence supporting the projected 40% reduction in unemployment.

```

Each question should reference the underlying requirement or issue.

---

# 12. Assessment lifecycle

Implement:

```text
draft
in_review
reviewed
stale

```

An assessment becomes stale when the guideline or application version used by the assessment is no longer the latest version.

The system must prevent final review/report generation from silently using stale source versions.

Show a clear UI warning:

> This assessment is stale because the guideline/application changed. Create a new assessment to continue.

---

# 13. Versioning

Preserve:

- guideline versions
- application versions
- assessment version/reference
- mapping versions
- reviewer decisions

Never overwrite historical source documents.

Each version should have:

- version number
- created timestamp
- content hash

Example:

```text
Guideline v1
Guideline v2

Application v1
Application v2

```

An assessment must remember exactly which versions were used.

---

# 14. Audit history

Maintain a complete decision history.

Record events such as:

- assessment created
- analysis started
- analysis completed
- mapping confirmed
- mapping corrected
- mapping rejected
- document status changed
- guideline version uploaded
- application version uploaded
- assessment marked stale
- report generated

Each audit event should include:

- timestamp
- event type
- entity
- entity ID
- actor
- before value if relevant
- after value if relevant
- note

Provide an Audit History UI.

---

# 15. Reviewed report

Generate a reviewed completeness report containing:

## Summary

- assessment name
- guideline version
- application version
- assessment status
- mandatory completion percentage

## Mandatory requirements

For every requirement:

- requirement
- status
- evidence
- guideline citation
- application citation
- reviewer decision

## Missing/weak items

List all unresolved items.

## Supporting documents

Show provided/missing/pending documents.

## Unsupported claims

Show unsupported claims.

## Clarification questions

Show questions requiring applicant clarification.

## Disclaimer

Include:

> This assessment is an evidence-based review of the supplied documents and is not a formal legal, regulatory, or funding-eligibility determination.

---

# 16. API design

Create clean REST APIs.

Suggested endpoints:

```text
GET    /api/health

POST   /api/guidelines
GET    /api/guidelines
GET    /api/guidelines/{id}
POST   /api/guidelines/{id}/versions

POST   /api/applications
GET    /api/applications
GET    /api/applications/{id}
POST   /api/applications/{id}/versions

POST   /api/assessments
GET    /api/assessments
GET    /api/assessments/{id}

POST   /api/assessments/{id}/analyze

GET    /api/assessments/{id}/requirements
GET    /api/assessments/{id}/mappings
GET    /api/assessments/{id}/summary

PATCH  /api/mappings/{id}/review

GET    /api/assessments/{id}/documents
PATCH  /api/documents/{id}

GET    /api/assessments/{id}/questions
GET    /api/assessments/{id}/claims

GET    /api/assessments/{id}/audit
GET    /api/assessments/{id}/report

```

Use proper HTTP status codes.

Return structured error responses.

---

# 17. Database design

Use normalized tables/models.

Recommended entities:

```text
Guideline
GuidelineVersion

Application
ApplicationVersion

Assessment

Requirement
Mapping
MappingDecision

SupportingDocument

UnsupportedClaim
ClarificationQuestion

AuditEvent

```

Relationships must preserve historical versions.

Use foreign keys.

Add timestamps.

Use indexes where useful.

---

# 18. Frontend design

Create a professional dashboard.

### Dashboard

Show:

```text
Assessment
Guideline
Application
Status

Mandatory completion: 75%

8 Mandatory
6 Satisfied
2 Missing
0 Weak
0 Ambiguous

Supporting documents: 3/5

Unsupported claims: 1

Pending review: 0

```

Use clear status badges.

---

# 19. Requirements page

Display requirements as cards/table.

Columns:

```text
Requirement
Priority
Status
Evidence
Guideline Citation
Application Citation
Review

```

Allow opening a requirement for detailed review.

---

# 20. Review experience

For each mapping provide:

```text
AI assessment
Evidence
Citation
Confidence
Reason

[Confirm]
[Correct]
[Reject]

```

Correction form:

```text
Status
Evidence
Citation
Reviewer note
Save

```

Make it obvious which values came from AI and which came from a human reviewer.

---

# 21. Documents page

Show:

```text
Document
Required
Status
Version
Actions

```

Allow changing status.

---

# 22. Audit page

Timeline-style UI:

```text
10:32 — Assessment created
10:34 — AI analysis completed
10:40 — Mapping REQ-005 corrected
10:41 — Budget document marked provided

```

---

# 23. Report page

Create a clean printable report.

Provide:

```text
Generate Reviewed Report

```

Do not call the result a certification.

---

# 24. AI architecture

Do not put AI logic directly inside API routes.

Use:

```text
app/
  ai/
    client.py
    prompts.py
    schemas.py
    pipeline.py

```

Pipeline:

```text
Guideline
   ↓
Requirement extraction
   ↓
Application evidence extraction
   ↓
Requirement mapping
   ↓
Unsupported claim detection
   ↓
Clarification generation
   ↓
Persist structured result

```

Use structured JSON outputs.

Validate all AI responses with Pydantic.

If AI returns malformed JSON:

1. log the error
2. retry once if appropriate
3. fall back to a safe error state
4. never corrupt assessment data

---

# 25. Prompt design

Create separate prompts for:

### Requirement extraction

Tell the model:

- use only supplied guideline
- extract explicit requirements
- distinguish mandatory/recommended
- preserve source wording
- provide exact section citations
- do not invent requirements

### Evidence mapping

Tell the model:

- use only supplied application
- map each requirement
- provide exact evidence
- cite application section
- distinguish missing/weak/ambiguous/satisfied
- never invent evidence

### Unsupported claims

Tell the model:

- identify factual claims requiring evidence
- compare against supplied supporting evidence
- do not use outside knowledge
- explain why evidence is insufficient

### Clarification generation

Tell the model:

- ask concise actionable questions
- derive questions only from identified gaps
- do not invent missing facts

---

# 26. Demo mode

The project MUST work without an API key.

Create a realistic demo dataset containing:

### Guideline

At least 8 mandatory requirements and 1 recommended requirement.

### Application

Enough evidence to demonstrate:

- satisfied
- missing
- weak
- ambiguous
- unsupported claim

### Supporting documents

At least 5 documents with a mixture of provided and missing statuses.

Create a demo endpoint or seed command.

Example:

```text
POST /api/demo

```

It should create a complete assessment that reviewers can immediately explore.

---

# 27. Testing

Write automated tests for:

### Backend

- health endpoint
- assessment creation
- requirement extraction persistence
- mapping persistence
- review confirmation
- correction
- rejection
- deterministic completeness calculation
- supporting document status
- unsupported claims
- version creation
- stale assessment detection
- stale review prevention
- audit history
- report generation

At least 10 meaningful backend tests.

### Frontend

If practical, add tests for:

- dashboard rendering
- review action
- status display
- stale warning

---

# 28. Security

Implement basic production hygiene:

- environment variables for secrets
- CORS configuration
- input validation
- file type/size validation
- no API keys in source code
- no uploaded documents committed to Git
- no database files committed to Git
- no node_modules committed to Git
- safe error messages
- basic request logging

Create:

```text
.env.example
.gitignore

```

---

# 29. Docker

Provide:

```text
backend/Dockerfile
frontend/Dockerfile
docker-compose.yml

```

Docker Compose should allow:

```bash
docker compose up --build

```

to start the application.

Document local non-Docker development as well.

---

# 30. README

Write your own concise README.

Include:

- project overview
- architecture
- features
- technology stack
- setup
- environment variables
- running locally
- running with Docker
- API overview
- test commands
- AI architecture
- deterministic scoring explanation
- versioning/staleness explanation
- responsible AI limitations

DO NOT copy the assessment's original problem statement or evaluation rubric into README.

---

# 31. UX requirements

The UI should feel like a real internal review product, not a coding demo.

Use:

- responsive layout
- clear navigation
- empty states
- loading states
- error states
- confirmation messages
- accessible buttons
- consistent status badges
- readable evidence citations
- clear reviewer-vs-AI distinction

Avoid excessive animations.

---

# 32. Error handling

Every important operation must handle:

- invalid input
- missing document
- malformed AI output
- AI unavailable
- stale assessment
- invalid review action
- missing required fields
- unsupported file type
- oversized file

Return useful errors such as:

```json
{
  "detail": "Assessment is stale; create a new assessment before reviewing."
}

```

---

# 33. Important implementation constraints

Do NOT:

- make legal/funding eligibility decisions
- claim formal compliance/certification
- use external grant databases
- fabricate evidence
- fabricate citations
- let the LLM calculate final completeness
- overwrite historical versions
- silently modify reviewer decisions
- silently ignore stale assessments

Do:

- preserve AI output
- preserve reviewer decisions
- make calculations deterministic
- make evidence traceable
- make uncertainty visible
- keep an audit trail

---

# 34. Final acceptance checklist

Before declaring the project complete, run:

### Backend

```bash
pytest -q

```

### Frontend

```bash
npm install
npm run build

```

### Docker

```bash
docker compose config

```

### Git

Verify:

```text
node_modules/     NOT tracked
__pycache__/      NOT tracked
*.pyc             NOT tracked
*.db              NOT tracked
.env              NOT tracked

```

---

# 35. Final deliverable

Produce a complete repository:

```text
grant-application-completeness-assistant/
│
├── backend/
│   ├── app/
│   │   ├── ai/
│   │   ├── api/
│   │   ├── models/
│   │   ├── services/
│   │   └── main.py
│   ├── tests/
│   ├── requirements.txt
│   └── Dockerfile
│
├── frontend/
│   ├── src/
│   ├── package.json
│   ├── Dockerfile
│   └── nginx.conf
│
├── docker-compose.yml
├── .env.example
├── .gitignore
└── README.md

```

Work incrementally.

First create the backend/database foundation and API contracts.

Then implement the deterministic assessment/review logic.

Then implement the AI abstraction and demo mode.

Then build the frontend.

Then integrate everything.

Then write tests.

Then perform a complete end-to-end verification.

Do not stop after generating scaffolding. The final repository must be runnable and demonstrable without an external AI key.