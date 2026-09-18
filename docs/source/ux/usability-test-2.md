# Usability Test Findings
## Story: Usability Test 2 – User and PM/Admin Scenarios

**Test Date:** 2026-09-16  
**Test Facilitators:** Finjas Bez, David Leuter, Anitta Pulikkathara  
**Participants:** One user and one PM/admin  
**Sprint:** Sprint 9

---

## 1. Test Goal

The goal of this usability test was to evaluate whether a user and a PM/admin can understand and use the current SprintStart onboarding and project management features.

The test focused on the currently implemented features:

- Chat and Buddy
- Access Management
- Board and PM Dashboard
- Onboarding Editor
- Starter Work
- Knowledge Base
- General navigation and interface behavior

---

## 3. Overall Summary

The two participants understood several central features and reacted especially positively to the Buddy, the project creation wizard, and the new onboarding graph. The main difficulties concerned unclear PM workflows, the Starter Work area, and some interactions in the onboarding editor. Additional usability and performance issues were observed across the chat, board, Knowledge Base, and general interface.

---

# Top 3 Successes

## Success 1: The Buddy was helpful and easy to understand

**Description:**
The participant found the Buddy useful and understood its core interaction patterns.

**Evidence:**
Links in the Buddy chat were received positively, and the meaning of locked content was understood immediately. Overall, the Buddy was described as helpful.

**Impact:**
The Buddy can support users effectively without requiring much additional explanation.

---

## Success 2: The new onboarding graph was visually convincing

**Description:**
The new graph-based onboarding design made a strong positive impression.

**Evidence:**
The participant described the graph as looking very good.

**Impact:**
The graph provides a promising visual foundation for understanding and managing onboarding paths.

---

## Success 3: Key configuration features were understandable

**Description:**
The participant understood the project creation wizard and the customizable dashboard.

**Evidence:**
The project creation wizard was rated positively, and the dashboard customization was easy to understand.

**Impact:**
PMs and admins can configure projects and dashboards with little initial guidance.

---

# Top 3 Pain Points

## Pain Point 1: The Buddy did not suggest Starter Work tasks

**Area:** Buddy and Starter Work
**Severity:** High

**Description:**
The Buddy did not suggest available Starter Work tasks when the participant asked what else they could do.

**Evidence:**
The Buddy still did not recommend suitable tasks after the participant asked explicitly and mentioned that the PM had requested it.

**Impact:**
New users may miss relevant starter tasks even when they actively ask the Buddy for more work.

---

## Pain Point 2: Important PM workflows were hard to discover

**Area:** PM Dashboard
**Severity:** High

**Description:**
Important actions, labels, and status information were not clear enough in the PM workflow.

**Evidence:**
The participant could not find the option to create a new project role, although this area has since been redesigned. Skills separated by commas and the meaning of “knowledge gaps” were unclear. Skip requests also could not be approved or rejected directly in the step details.

**Impact:**
PMs may struggle to understand project information or need unnecessary navigation to complete common management tasks.

---

## Pain Point 3: The onboarding editor contained unclear and inconsistent interactions

**Area:** Onboarding Editor
**Severity:** Medium

**Description:**
Several graph and knowledge-question interactions did not clearly communicate their purpose, state, or expected behavior.

**Evidence:**
Sample answers could be copied directly into knowledge questions, feedback was not visually distinct, and skip requests were not visible on graph nodes. Arrow movement, zoom controls, the “Questions” tab, and the placement of “Add Step” also caused confusion.

**Impact:**
These inconsistencies can make onboarding paths harder to edit and weaken the reliability of knowledge checks.

---

# Final Note

Further observations included chat formatting and feedback issues, missing message queueing, input lag during longer Buddy conversations, delayed board updates, slow Knowledge Base summaries, and inconsistent labels, icons, input states, and keyboard navigation. These points should be reviewed separately during implementation planning.
