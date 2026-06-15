# Usability Test Findings
## Story: Usability Test 1 – Student New-Joiner Scenario

**Issue:** #125  
**Test Date:** 2026-06-02  
**UX Lead:** Finjas Bez  
**Participant:** Brian Karle  
**Sprint:** Sprint 1

---

## 1. Test Goal

The goal of this usability test was to evaluate whether a student new-joiner can understand and use the current SprintStart onboarding flow.

The test focused on the currently implemented features:

- Login
- Role Selection
- Markdown Upload
- Onboarding Path

---

## 3. Overall Summary


The participant was able to complete the main onboarding flow from login to viewing the onboarding path. The general purpose of the application was understandable, but some parts of the flow still need clearer guidance. Since the chat feature was not available, the full intended onboarding scenario could not be tested.

---

# Top 3 Successes

## Success 1: Clear understanding of the Knowledge Base

**Description:**
The participant immediately understood what to do in the Knowledge Base section.

**Evidence:**
The tester knew directly that the Knowledge Base is used to upload or manage project-related markdown documentation.

**Impact:**
This is positive because new developers can quickly understand where project knowledge is stored and how they can contribute relevant information without needing additional explanation.

---

## Success 2: Smooth checkout process in the Role Selection Wizard

**Description:**
The Role Selection Wizard was easy to follow and the participant was able to complete the selection process successfully.

**Evidence:**
The tester completed the selection wizard without major confusion and was able to proceed through the steps.

**Impact:**
This is positive because the role selection is an important first step in personalizing the onboarding experience for new developers.

---

## Success 3: Progress bars supported orientation

**Description:**
The progress bars helped the participant understand their current progress within the onboarding flow.

**Evidence:**
The tester reacted positively to the progress bars and was able to see how far they had progressed.

**Impact:**
This improves the user experience because new users get a clear sense of progress, structure, and completion while going through the onboarding path.

---

# Top 3 Pain Points

## Pain Point 1: No clear visual feedback when all subtasks are completed

**Area:** Onboarding Path
**Severity:** Medium

**Description:**
It was not clear enough when a main step should be considered completed after all subtasks were done.

**Evidence:**  
During the test, the participant was confused that he had to manually mark the main step as finished, even though all subtasks were completed.

**Impact:**  
This can confuse new users because they may see a section of the onboarding path as completed, even though it is not officially marked as completed.

**Recommended Improvement:**  
Add a clear visual highlight to the "Finish main task" button when all subtasks are finished, or automatically mark the main task as finished.

**Linked Issue:**
[Automatic main step](https://github.com/SprintStartProject/sprintstart-frontend/issues/42)

---

## Pain Point 2: Unclear wording for “Experience”

**Area:** Role Selection
**Severity:** Low

**Description:**
The participant did not immediately understand what was meant by “Experience” in the role selection process.

**Evidence:**
The tester was unsure which experience level to select because the label was not specific enough.

**Impact:**
This can lead to incorrect role configuration and may result in an onboarding path that does not fully match the user’s actual skill level.

**Recommended Improvement:**
Change the wording from “Experience” to a clearer question, such as: “How experienced are you in your role?”

**Linked Issue:**
[Unclear Wording](https://github.com/SprintStartProject/sprintstart-frontend/issues/43)

---

## Pain Point 3: Upload button is too prominent compared to the repository content

**Area:** Knowledge Base 
**Severity:** Low

**Description:**  
The upload button in the Knowledge Base section appears too large and visually dominant compared to the actual knowledge repository content.

**Evidence:**  
During the test, the participant noticed that the upload button took too much visual attention in relation to the repository content.

**Impact:**  
This can distract new users from the main Knowledge Base content and may make the layout feel visually unbalanced.

**Recommended Improvement:**  
Reduce the size or visual emphasis of the upload button so that it better fits the hierarchy of the Knowledge Base page and supports the repository content instead of dominating it.

**Linked Issue:**  
[Too prominent upload button](https://github.com/SprintStartProject/sprintstart-frontend/issues/44)

---

# Final Note

The full intended scenario could not be tested because the chat and AI question-answering functionality was not available at this point in the current sprint. The results therefore only cover the implemented onboarding flow up to the onboarding path.