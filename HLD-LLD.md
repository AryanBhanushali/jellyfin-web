---
name: generate-design-docs
description: Analyzes this repository and generates/updates a High-Level Design (HLD) document and a Low-Level Design (LLD) document under docs/, written for an Enterprise Architect audience with a strong focus on security and data provenance.
---

# Generate Design Documents (HLD + LLD)

You are acting as a documentation engineer. Your job is to keep this repository's
architecture documentation accurate and current by analyzing the current state of
the code and updating two documents:

- `docs/HLD.md` — High-Level Design
- `docs/LLD.md` — Low-Level Design

## Audience

The reader is an **Enterprise Architect** who needs enough detail to review and
approve the design. Prioritize:
- Security posture (authn/authz, secrets handling, threat surface)
- Data provenance (what data exists, where it comes from, where it is stored,
  where it travels, retention)
- Clear architecture boundaries and integrations

Prose should be concise, technical, and vendor-neutral. Use bullet lists and small
tables over long paragraphs. No marketing language.

## Hard Rules

1. **Never invent facts.** Every technical claim must be grounded in the actual
   code, config files, README, IaC, or CI files in this repository. If something
   cannot be determined from the repo, write "Not determined from repository"
   rather than guessing.
2. **Update, don't rewrite.** If `docs/HLD.md` or `docs/LLD.md` already exist,
   only change the sections whose underlying subject matter has actually changed
   since the last version. Leave unaffected sections untouched. If a document
   does not exist yet, generate it in full (bootstrap mode).
3. **Never commit secrets.** If you encounter what looks like a real credential
   or key in a config/env file, redact it in the doc (e.g. `[REDACTED]`) and add
   a note under Security flagging it for manual review. Do not reproduce it.
4. **Write scope is `docs/` only.** Do not modify application source code,
   config, or infra files as part of this task.
5. **Diagrams use Mermaid.** Any architecture or flow diagram must be valid
   Mermaid syntax embedded directly in the markdown (fenced ```mermaid block),
   not an external image. Follow the Mermaid Syntax Rules below exactly —
   invalid syntax breaks rendering entirely.

## Mermaid Syntax Rules (avoid rendering errors)

Mermaid is strict, and GitHub's renderer gives no partial credit — one bad
character anywhere in the diagram breaks the entire diagram. Apply every rule
below, not just the ones that have caused failures before.

### A. Node labels

1. **Never start a node label with a shape-reserved character.** Characters
   like `/`, `\`, `(`, `[`, `{`, `>` immediately after the opening bracket are
   reserved for special node shapes (trapezoid, parallelogram, rounded,
   subroutine, rhombus, etc.). `[/auth,/users,/tasks]` is invalid for this
   reason. If your text needs to start with one of these characters, wrap the
   whole label in double quotes: `["/auth, /users, /tasks"]`.
2. **Wrap any label containing punctuation in double quotes.** This includes
   commas, slashes, parentheses, colons, semicolons, ampersands, and angle
   brackets. Example: `A["API Layer: /auth, /users, /tasks"]`.
3. **Never put a literal double quote inside a double-quoted label.** Mermaid
   has no escape character for `"` inside a quoted string. Rephrase instead
   of quoting text within a label (e.g. write `the config value` instead of
   `"config"`).
4. **Never use a raw line break inside a label.** Use `<br/>` for a line
   break within a node label, never an actual newline character.
5. **Don't redefine a node's shape twice.** Once a node ID has been given a
   shape (e.g. `A["Text"]`), later references should use just the ID (`A`),
   not redeclare it with a different bracket type (e.g. `A(("Text"))`) —
   conflicting shape declarations for the same ID cause errors.
6. **Node IDs themselves must not contain spaces or punctuation.** The ID is
   the part before the bracket; only the label text inside the brackets can
   contain spaces. Bad: `My Node["Text"]`. Good: `MyNode["Text"]`.

### B. Edge labels

7. **The same punctuation rule applies to edge labels** (the text between
   pipes in `-->|label|`), and it's easy to fix node labels while forgetting
   this. `-->|HTTPS (browser)|` is invalid; use `-->|"HTTPS (browser)"|`, or
   better, simplify to `-->|HTTPS|` and move the nuance into surrounding
   prose.
8. **Use a real arrow syntax.** In `flowchart` diagrams, valid arrows are
   `-->`, `---`, `-.->`, `==>`, and similar two/three-character forms. A
   single-character arrow like `->` is invalid in flowchart syntax.
9. **Keep arrow style consistent within one diagram** unless the style
   difference is intentional (e.g. dashed for async, solid for sync) — and if
   so, note that convention once in the surrounding prose.

### C. Reserved words

10. **Never use a Mermaid reserved word as a node ID or subgraph ID.** This
    includes `end`, `graph`, `flowchart`, `subgraph`, `class`, `style`,
    `click`, `direction`. `end` is the most common accidental collision (e.g.
    a node meant to represent "Endpoint" or "End User" abbreviated as `end`
    breaks the parser). Use a different ID like `EndUser` or `Endpoint`, and
    put "End" only in the label text if needed.

### D. Structure

11. **One node or edge statement per line.** Do not chain multiple
    declarations with commas on a single line.
12. **Comments must be on their own line**, using `%%` at the start of the
    line — never appended after a node/edge definition on the same line.
13. **Declare a consistent direction once** (`flowchart TB`, `TD`, `LR`, `BT`,
    or `RL`) at the top; don't mix direction keywords mid-diagram.
14. **classDef and class names must be simple identifiers** (letters, digits,
    underscores only) — no spaces or punctuation in a `classDef` name.

### E. Sequence diagrams (used in the LLD's workflow diagrams)

15. **Participant names with spaces need an alias**: use
    `participant WC as "Web Client"`, then refer to `WC` in messages — never
    a bare multi-word name directly in a message line.
16. **Never use a reserved word as a participant alias**, including `end`,
    `loop`, `alt`, `opt`, `par`, `rect`, `note`, `activate`, `deactivate`.
17. **Message text (after the colon) must not contain an unescaped colon.**
    The first colon in a message line separates the arrow from the message
    text; a second literal colon later in the same line can break parsing.
    Rephrase to avoid a second colon, or use the HTML entity `#58;`.
18. **Use valid arrow types only**: `->>`, `-->>`, `-)`, `--)`, `-x`, `--x`,
    or `->`/`-->` for the two solid/dashed base forms — don't invent
    variants.
19. **Every `activate` must have a matching `deactivate`** for the same
    participant, in the correct order — unbalanced activation blocks break
    rendering.

### F. Final check

20. **Before finalizing any diagram, re-parse it mentally line by line** and
    confirm: every node label and edge label is either a bare word/short
    phrase with zero punctuation, or fully wrapped in double quotes; no
    reserved words are used as IDs; arrows are valid two/three-character
    forms; and (for sequence diagrams) every alias is declared before use and
    every activation is closed.

Example of a safe layered flowchart opening:

```mermaid
flowchart TB
    subgraph Client
        UI["Web UI"]
    end
    subgraph API
        Gateway["API Gateway: /auth, /users, /tasks"]
    end
    UI -->|HTTPS| Gateway
```

Example of a safe sequence diagram opening:

```mermaid
sequenceDiagram
    participant WC as "Web Client"
    participant API as "API Gateway"
    WC->>API: Submit request
    activate API
    API-->>WC: Return response
    deactivate API
```

## HLD Structure (`docs/HLD.md`)

Use this section outline as the default. Add sections the repo clearly warrants
(e.g. a "Message Queue" section if Kafka/SQS is present) and omit sections that
don't apply (e.g. skip "API Surface" if this repo exposes no API). Note any
additions/removals at the top of the Change Log.

1. Title & Metadata (repo name, last updated date, doc owner)
2. Executive Overview
3. Objective
4. Architecture Description (with an embedded Mermaid `flowchart TB` diagram,
   using layered subgraphs where they apply: Client / API / Orchestration /
   Core / Data, plus external systems such as Auth or third-party APIs on the
   side; label edges with protocol, e.g. HTTPS, gRPC, JWT)
5. Core Workflows
6. Data Flow
7. Key Features
8. Infrastructure & Deployment Overview
9. Deployment Strategy
10. Data Protection (data in transit, data at rest, secrets management, any
    LLM/third-party data sharing, logging, retention policy)
11. Security Requirements (authentication/authorization model, threat
    considerations, dependency posture — e.g. known-vulnerable dependencies if
    detectable)
12. Integrations (each third-party or external connection this repo makes:
    what it connects to, why, and how it authenticates — e.g. PAT, OAuth,
    GitHub App, API key)
13. Environment Variables & Secrets Inventory (derived from `.env.example`,
    `.envrc`, Kubernetes manifests, etc. — names and purposes only, never
    values)
14. Change Log (append a dated entry each time this doc is updated,
    summarizing what changed and why)

## LLD Structure (`docs/LLD.md`)

1. Title & Metadata
2. Module/Component Breakdown (one subsection per major module or package,
   its responsibility, and its public interface)
3. Key Classes / Functions (only the architecturally significant ones — not
   an exhaustive dump; note purpose, inputs/outputs, and important
   side effects)
4. Data Models / Schemas (database tables, request/response schemas, or
   equivalent — include field names and types where determinable)
5. Sequence Diagrams for the 1–3 most important workflows (Mermaid
   `sequenceDiagram`)
6. Error Handling & Retry Behavior
7. Configuration & Environment-Specific Behavior
8. Known Limitations / Technical Debt (only if evident from TODOs, comments,
   or clearly incomplete implementations — do not speculate)
9. Change Log

## Process

1. Read the current `docs/HLD.md` and `docs/LLD.md` if they exist, along with
   any existing Change Log entries, to understand what was last documented.
2. Inventory the repository: file tree, README, dependency manifests
   (`package.json`, `pyproject.toml`, `go.mod`, etc.), Dockerfiles, CI configs,
   IaC, and `.env.example`/similar files.
3. Identify what is new, changed, or removed relative to what's currently
   documented.
4. Update only the affected sections of `docs/HLD.md` and `docs/LLD.md`
   following the structures above. If bootstrapping (no existing docs),
   generate both in full.
5. Append a dated Change Log entry in both documents summarizing what was
   updated and why.
6. Write the files to disk. Create the `docs/` folder if it does not exist,
   and write/update `docs/HLD.md` and `docs/LLD.md` directly in the working
   tree.
7. If there are no actual changes to `docs/HLD.md` or `docs/LLD.md` compared
   to what's already committed, stop here — do not create a branch or PR.
8. If there are changes, open a pull request:
   - Create a new branch named `docs/auto-hld-<YYYYMMDD-HHMM>` off the
     default branch.
   - Commit the changed files with a message like
     `docs: automated HLD/LLD update <date>`.
   - Push the branch and open a pull request against the default branch,
     titled `docs(hld): automated update <date>`, with the label
     `automerge`, and a body summarizing what changed in each document.
   - Do not push directly to the default branch under any circumstance —
     changes must always go through a PR.
