# ADR-013: Wiki structure and documentation framework

| Field | Value |
|---|---|
| **Status** | `Proposed` |
| **Date** | 2026-06-12 |
| **Deciders** | Docs / Knowledge Lead |
| **Supersedes** | – |
| **Superseded by** | – |

---

## Context

We have started to reach a point in development where documentation becomes more and more important due to the codebase increasing in size in all repositories with each sprint.
A growing amount of code requires information on how the code is structured, how to contribute to the code, why certain (architectural) decisions have been made, etc. This leads
to the problem of having to decide on how to structure the documentation in the Wiki, as it is important that required information can be found quickly.

It is important to make this decision now as it will be very difficult to restructure the wiki once it has grown too large. Furthermore, sticking to a specific documentation
framework early on can facilitate the process of familiarizing with the way documentation is handled.

## Decision Drivers

- Categorization should be unambigious and clear; as few conflicts as possible
- Documents of similar type should be held in one folder
- Intuitive and simple structure, nothing too complicated

## Considered Options

- **Option A** – Organize by repository
  
  ```text
  Wiki
  ├── Frontend
  │   ├── ADRs
  │   ├── Dev manuals
  │   ├── Usability tests
  │   └── ...
  │ 
  ├── Backend
  │   ├── ADRs
  │   ├── Dev manuals
  │   ├── Architecture
  │   └── ...
  │
  ├── AI
  │   ├── ADRs
  │   ├── Dev manuals
  │   └── ...
  │
  └── ...
  ```

- **Option B** – Organize by topic

  ```text
  Wiki
  ├── Project
  │   ├── Working Agreements
  │   ├── Definition of done
  │   └── ...
  │ 
  ├── Architecture
  │   ├── ADRs
  │   ├── Diagrams
  │   └── ...
  │
  ├── Dev Manuals
  │   ├── Getting started
  │   ├── Dev manuals
  │   └── ...
  │
  ├── Guidelines
  │   ├── Conventions
  │   └── ...
  │
  └── ...
  ```
  
- **Option C** – Diátaxis framework

  ```text
  Wiki
  ├── Tutorials
  │   └── ...
  │ 
  ├── How-to guides
  │   └── ...
  │
  ├── References
  │   └── ...
  │
  ├── Explanations
  │   └── ...
  │
  └── ...
  ```

  For more information, see [diataxis-introduction](../diataxis-introduction.md).

## Decision

**Chosen option: Option C** – Diátaxis

## Rationale

All things considered, Diátaxis appears to be the best compromise between the considered options. Alternative options have overwhelming drawbacks, and combinations or
hybrid versions of said options only accumulates downsides instead of resolving them. Diátaxis, on the other hand, is actually able to offer solutions: documents of
the same type can all be found in one place and the general structure is definite while at the same type expandable by just adding subfolders. The downside is that
it is hard to estimate how many subfolders will actually be needed and how well they will be distributed among the top-level folders, still, this is a problem all
of the options face in one way or another, and with diátaxis being a generally well-reviewed framework, we can expect this problem to be neglectable.

## Pros and Cons of the Options

### Option A – Organize by repository

- ✅ Intuitive: easy to classify and easy to find
- ❌ Some documents belong to more than one repository or no repository at all → exceptions needed
- ❌ Documents that belong to the same type get split up (ADRs, ...)

### Option B – Organize by topic

- ✅ Centralized knowledge: document types that apply to all repositories are all in one place
- ❌ Might get out of hand; it is not determined how many categories actually exist
- ❌ Team-specific knowledge is harder to find

### Option C – Diátaxis framework

- ✅ Structured knowledge: very good if I know what kind of information I'm looking for
- ✅ Requires thinking about knowledge in a deeper way; knowing the intended purpose of a document can notably increase docs quality
- ✅ Has beenadopted by multiple projects and reviewed positively
- ❌ Dependent on estimation of other teammates as some documents might be more difficult to classify
- ❌ Rather abstract depiction of knowledge; can lead to confusion, team needs to get used to it
- ❌ Not created specifically for software projects

## Consequences

**Positive:**
- Documentation will get much easier and more well-arranged in later stages of development
- Getting used to more professional sorts of docs / knowledge management

**Negative / Trade-offs:**
- Needs time to get into; conflicts and confusions likely at the start
- Estimation might differ from actual wiki size in a later stage 

**Follow-up actions:**
- [ ] Mark status as 'accepted'
- [ ] Update index.md (see below)
- [ ] Add link for reference document on how the wiki ist structured

## Links

- [ADR Index](index.md)
