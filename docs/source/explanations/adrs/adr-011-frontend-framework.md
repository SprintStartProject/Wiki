# ADR-005: Frontend Framework

| Field | Value         |
|---|---------------|
| **Status** | `Accepted`    |
| **Date** | 2026-05-19    |
| **Deciders** | Frontend Team |
| **Supersedes** | –             |
| **Superseded by** | –             |

---

## Context

In order to start developing the frontend, the team needs to decide which of the various existing frameworks to use. Given the impact on architecture, workflow, and other fundamental
aspects, this should be considered an important decision. Due to the team's experience being mostly limited to JavaScript, we decided to consider only JavaScript
frameworks as viable options.

## Decision Drivers

- Support for component-based architecture
- Support for complex, interactive states
- Extensive ecosystem / library support
- Builds on team's previous experience / available tutorials to get started quickly

## Considered Options

- **Option A** – Vue.js
- **Option B** – React
- **Option C** – Angular

## Decision

**Chosen option: React (+ TypeScript)**

## Rationale

React provides an extensive support for reusable components, animations, and third-party-libraries, enabling fast and efficient implementation of interactive user interfaces.
The same applies to the support for state management and component-based architectures. Furthermore, experience with React already exists within the team. Since React is commonly
used with TypeScript, we decided to use it as our script language instead of JavaScript.

## Pros and Cons of the Options

### Option A – Vue.js

- ✅ Clear separation of logic, template, and styling
- ✅ Existing team experience
- ❌ Smaller ecosystem (compared to React)
- ❌ Fewer integrations and third-party libraries

### Option B – React

- ✅ Suited for highly interactive and dynamic user interfaces
- ✅ Extensive library support and ecosystem
- ❌ Logic and template in one component
- ❌ Dependent on external libraries for common tasks

### Option C - Angular
- ✅ Excellent TypeScript integration
- ✅ Strong architectural conventions and structure
- ❌ Less flexibility due to strong framework conventions
- ❌ High complexity and learning curve

## Consequences

**Positive:**
- Faster implementation of tasks of interactive components and animations
- Easier integration of modern web- and AI-based technologies
- Reduced amount of runtime errors due to type safety via TypeScript

**Negative / Trade-offs:**
- Some team members will have to catch up on missing knowledge

**Follow-up actions:**
- [ ] Update ADR status from `Proposed` to `Accepted`

## Links

- [ADR Index](index.md)
- [sprintstart-frontend](https://github.com/SprintStartProject/sprintstart-frontend)
