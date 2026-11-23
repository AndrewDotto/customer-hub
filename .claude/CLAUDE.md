# Strawberry Moon Project — Agent Guide

This is a **Strawberry Moon** project. Strawberry Moon is an AI-powered product management platform that organises product strategy and execution through a structured document hierarchy. All documents are **Markdown files** with specific naming conventions and parent-child relationships.

This guide describes the document model. Engineering conventions for a particular repository, where they exist, live in `.claude/rules/` (Claude Code loads them automatically) and apply alongside it. The code is the source of truth for how the product behaves; this guide is the source of truth for how documents are named, numbered and related.

---

## Document Hierarchy

Strawberry Moon uses two workstreams: **Product** (strategy) and **Execution**. There is one project type and one press release type.

### Product — Strategy Hierarchy (top-down)

```
Market Analysis (.market.md)
 └── Product Vision (.product.md)            ← 1:many under Market
      ├── Architecture (.architecture.md)   ← 0:many under Product Vision
      └── UX Design (.ux.md)               ← 0:many under Product Vision
```

### Execution — Delivery Hierarchy (top-down)

```
Press Release (.pr.md)
 ├── Epic (.epic.md)            ← 1:many under Press Release; the Product
 │    ├── User Story (.story.md)   function's unit of work
 │    │    ← 1:many under Epic
 │    └── Task (.task.md)       ← Spec Driven mode only: 0:many under Epic,
 │                                 in place of User Stories; no Subtasks
 │
 ├── Task (.task.md)            ← 1:many under Press Release, sibling of Epic;
 │    └── Subtask (.subtask.md)    work owned by another function
 │         ← 1:many under Task     (Engineering, Marketing, Sales, ...)
 │
 └── Change (.change.md)        ← 0:many under Press Release, in a Changes/
                                   folder; a bug or a change request
```

Epic and Task are **siblings under the same Press Release** and are treated identically: a press release commonly holds both, they are displayed together in one list, and they are ordered purely by their shared numbering. The difference between an Epic and a Task is which organisational function owns the work. Both carry the same MoSCoW priority, T-shirt size and status fields, and both carry `function` in front matter (an Epic defaults to `Product`).

In **Spec Driven mode** (an organisation setting) an Epic is a complete functional specification and gets no new User Stories. It can instead be broken down into **Tasks under the Epic**. Such a Task takes the place a User Story would: it links to its Epic, not the press release, is numbered under the Epic, rolls up into the Epic's status like a story, and has no Subtasks. Tasks can still sit directly under the press release beside its Epics too.

A press release's **launch work** (Marketing, Sales and the other go-to-market functions) is planned in the Launch tab. A Task or Subtask created there carries `launch: true`, starts with a go-to-market function rather than Engineering, and shows in the Launch tab only, never in Planning, Kanban or the press release's Work list. Older non-Engineering Tasks created in Planning have no flag and show in both. A product can also launch a press release of another product that is referenced into it (the Portfolio's Reference releases): its launch Tasks are stored in the launching product, linked to that press release without being children of it, and numbered among that product's tasks for it.

A **Change** is the unit of incremental work: a bug or a change request with a **Before** section (current behaviour), an **After** section (required behaviour), a triage justification, a priority (P1 critical, P2 important, P3 nice to have) and a type (`bug` or `change`). Changes live in a `Changes/` folder under their press release, or at the top of the Execution folder when they have no press release yet.

Press Releases are grouped by **cadence period** (quarters such as Q1, Q2, or halves such as H1, H2).

A Press Release can link back to a Product Vision document. The document-creation UI requires that link whenever the project already has a Product Vision document, so in practice Product and Execution form **one connected chain**: Market Analysis → Product Vision → Press Release → Epic/Task/Change → User Story/Subtask (or, in Spec Driven mode, a Task under an Epic). A Press Release created before any Product Vision document exists has no parent.

`initiative-press-release` (`.ipr.md`) is a category that exists only for documents created before press releases were unified. New press releases are never created in it; its existing Task and Subtask children keep working unchanged.

Tasks, Subtasks and `.ipr.md` documents live in Strawberry Moon only. They are never pushed to a connected repository and are not in its status index, so a coding agent reads them through the API.

### Initiatives and Business Cases

Press releases can be grouped into an **initiative**: a multi-quarter goal that may span several products. Initiatives are not documents; they are a grouping shown in the Portfolio as one box spanning its quarters in each product's swim lane.

An initiative can have one **Business Case** (`.business-case.md`), created from the product strategy business-case template (the initiative, costs and benefits, and the verdict) and edited with the product strategy assistant like any other document. Its press releases are not written into it; the business case view lists them below the document, grouped by period and product. Its KPIs live in its front matter by the same convention as a press release's (a target plus `actual_`, `start_` and `progression_` companions). It is stored in the product of the initiative's earliest release but belongs to the initiative: it is opened from the initiative, not from the product's document tree, and is never pushed to GitHub.

---

## File Naming Conventions

All documents use the `.md` extension with a **dual-extension** pattern that identifies the category:

| Category                 | Extension           | Example                                  |
|--------------------------|---------------------|------------------------------------------|
| Market Analysis          | `.market.md`        | `housing-in-uk.market.md`                |
| Product Vision           | `.product.md`       | `analytics-platform.product.md`          |
| Architecture             | `.architecture.md`  | `system.architecture.md`                 |
| UX Design                | `.ux.md`            | `dashboard.ux.md`                        |
| Press Release            | `.pr.md`            | `v2-launch.pr.md`                        |
| Epic                     | `.epic.md`          | `1.user-onboarding.epic.md`              |
| User Story               | `.story.md`         | `1.1.signup-flow.story.md`               |
| Task                     | `.task.md`          | `2.pricing-engine.task.md`               |
| Subtask                  | `.subtask.md`       | `2.1.api-work.subtask.md`                |
| Change / Bug             | `.change.md`        | `Changes/12.fix-login-overlap.change.md` |
| Roadmap                  | `.roadmap.md`       | `2025-roadmap.roadmap.md`                |
| Initiative Press Release | `.ipr.md`           | `q3-initiative.ipr.md`                   |
| Business Case            | `.business-case.md` | `payroll-growth.business-case.md`        |

Product Vision documents created before `.product.md` keep their older `.prd.md` name.

### Execution Numbering

Execution documents carry **sequential numbering prefixes** that drive sort order everywhere: the sidebar tree, Kanban, Canvas and the press release's "Work" list. Display order is the numbering alone; nothing is grouped by category. Epic and Task **share one numbering sequence** under their press release, so a press release can hold Epic 1, Task 2, Epic 3, Task 4 in whatever order they were created or reordered:

- Epic/Task under one press release: `1.name.epic.md`, `2.name.task.md`, `3.name.epic.md`, ...
- Stories under Epic 1: `1.1.name.story.md`, `1.2.name.story.md`
- Stories under Epic 3: `3.1.name.story.md`, `3.2.name.story.md`
- Subtasks under Task 2: `2.1.name.subtask.md`, `2.2.name.subtask.md`
- Tasks under Epic 3 in Spec Driven mode: `3.1.name.task.md`, `3.2.name.task.md`, in one sequence with any stories under Epic 3
- Changes number independently within their `Changes/` folder: `1.name.change.md`, `2.name.change.md`

When creating a new Task or Epic, the next number continues the combined sequence: it must clear every existing Epic **and** Task under the same press release. Likewise a new Story or Task under an Epic must clear every existing Story **and** Task under that Epic.

Opening a Press Release shows its Epics and Tasks together in one list titled **Work**, sorted by that shared numbering. Each row shows its number and its title.

### Titles

A file name is a storage path, not a title. Everywhere a document is shown (the Portfolio, the trees, the Work list, Kanban, Canvas, chat and exports) it is named by its title, chosen in this order:

1. `title:` in its front matter, written when the document is created or renamed;
2. its first `# Heading`, unless that only names the document type (`# Press Release`, `# User Story`);
3. its file name with the number, extension and dashes removed.

Renaming a document sets `title:` and leaves the file name, and so its GitHub path and links, unchanged. When you create a document, give it a short `title:` of three to eight words.

---

## YAML Front Matter

Epics, User Stories, Tasks, Subtasks and Changes use YAML front matter for metadata. Press Releases use `status`, `targetDate`, `period` and `tshirt`, and hold their KPIs there too.

### Reference Key (Epics, Tasks, Changes and Press Releases)

`key:` holds the document's reference key, such as `PAY-12`: the project's key prefix and a number from one sequence shared by those four categories across the project. The platform issues it and it never changes, so do not write or edit `key:`. A changed or removed key is put back on save, and a key typed into a new document is discarded unless it is a number this project has already issued and no other document holds. A project with no key prefix issues no keys, and an older document may not have one yet. User Stories, Subtasks, `.ipr.md` documents and Product documents have no key.

### Status Values

| Status        | Meaning                                  |
|---------------|------------------------------------------|
| `Draft`       | Initial creation, not reviewed           |
| `Ready`       | Approved for development                 |
| `In Progress` | Being worked on                          |
| `Review`      | Implementation complete, awaiting review |
| `Done`        | Accepted; set by a person                |
| `Blocked`     | Blocked by a dependency or a question    |
| `Won't Fix`   | Rejected (changes only)                  |

Through the status API, Press Releases and Product documents are set to `Draft` or `Ready` only. A Ready Press Release moves to `In Progress` on its own when one of its Epics does.

A person marks a press release `Done`, and the database refuses it until every Epic and Task directly under it, `launch: true` Tasks included, is `Done` with no story, task or subtask under them still open. Changes do not count, nor do launch Tasks another product created for it. A press release with nothing under it can be marked `Done`. The check runs only on the move into `Done`, so reopening a child later changes nothing. Inbound GitHub and Jira changes never move an existing press release into `Done`: the rest of the file applies and its status is kept. Finished work alone never makes a press release `Done`.

### MoSCoW Priority (Epics and Tasks)

Must, Should, Could, Will Not

### T-Shirt Sizing (Epics, Tasks and Press Releases)

XS, S, M, L, XL

### Function (Epics, Tasks and Subtasks)

Which organisational function owns the work: `Product`, `Engineering`, `Marketing`, `Sales`, or any other function the organisation has defined. An Epic defaults to `Product` when unset; this ownership is what distinguishes an Epic from a Task.

### KPIs and reserved names (Press Releases and Business Cases)

On a Press Release or Business Case, a front matter field with a numeric value is read as a KPI target unless its name is reserved. `actual_`, `start_`, `kpi_color_`, `kpi_combine_` and `progression_` fields are that KPI's companions. Reserved names, such as `status`, `tshirt`, `targetDate`, `period`, `priority`, `key` and `launch`, are never KPIs. Add a numeric field to these documents only when it is a KPI.

### Change fields (Changes only)

`priority` (1, 2 or 3, shown as P1 to P3) and `type` (`bug` or `change`). The body carries `## Before`, `## After` and `## Triage Justification` sections.

---

## AI Assistants

Each document type has its own AI assistant, with the capabilities and rules for that type. The assistants have no names and no personalities: they never introduce themselves or refer to themselves by a name, and start with the work the user asked for. Each assistant's definition (capabilities, behaviour rules and document flow) is a file in `.strawberrymoon/`; the upper-case file names are stable identifiers, not names. An organisation can replace any definition with its own.

| Assistant | Documents | Definition |
|-----------|-----------|------------|
| Market analysis | Market Analysis | `.strawberrymoon/SCOUT.agent.md` |
| Product strategy | Product Vision, Business Case | `.strawberrymoon/SONAR.agent.md` |
| Architecture | Architecture | `.strawberrymoon/COMPASS.agent.md` |
| UX design | UX Design | `.strawberrymoon/AURA.agent.md` |
| Planning | Press Release, Epic, User Story, Task, Subtask, Change | `.strawberrymoon/FLASH.agent.md` |
| The coding agent | Implements changes, stories and epics in code | `.strawberrymoon/CADE.agent.md` |

If you are acting as a coding agent, read `.strawberrymoon/CADE.agent.md`: it holds the status protocol the platform depends on, how to choose what to work on, and where to find context.
