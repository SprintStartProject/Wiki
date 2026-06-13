# Frontend Documentation Guidelines

## Purpose

This document defines the documentation standards for the React and TypeScript frontend codebase.

The goal is to improve maintainability, readability, and onboarding efficiency while avoiding unnecessary documentation overhead. Documentation should provide additional context where code alone is not sufficient to explain intent or behavior.

---

## General Principles

### Document the "Why", not the "What"

Comments should explain the purpose, reasoning, or business context behind the code.

**Good**

```tsx
/**
 * The onboarding path is loaded only after the user profile
 * becomes available because the backend requires the user ID.
 */
```

**Bad**

```tsx
// Set loading to true
setLoading(true);
```

The code itself already explains what happens.

---

### Keep Documentation Up-to-Date

Documentation must be updated whenever the documented behavior changes.

Outdated documentation is considered worse than no documentation.

---

### Use TSDoc/JSDoc Style

All documentation comments should follow the standard TypeScript documentation format.

```tsx
/**
 * Description of the component or function.
 */
```

---

## Components

### Large Page and View Components

All page-level or view-level components should contain a documentation block describing their responsibility.

**Examples**

```tsx
/**
 * Displays the onboarding path for the authenticated user.
 *
 * The path depends on the user's selected working area and is loaded
 * from the backend based on the current user ID.
 */
export function OnboardingView() {
  // ...
}
```

```tsx
/**
 * Allows users to select their working area during the onboarding process.
 *
 * The selected role is stored in the backend and used to generate
 * a personalized onboarding path.
 */
export function RoleSelectionView() {
  // ...
}
```

---

### Reusable Components

Reusable components should be documented when their purpose is not immediately obvious.

**Example**

```tsx
/**
 * Displays a summary card for an onboarding task.
 */
export function TaskCard() {
  // ...
}
```

Small presentational components with self-explanatory names usually do not require additional documentation.

---

## Props Documentation

Props should be documented when:

* The component is reused in multiple places.
* The meaning of a prop is not obvious.
* The prop influences complex behavior.
* The prop contains callback functions.

**Example**

```tsx
type TaskCardProps = {
  /**
   * Title displayed inside the task card.
   */
  title: string;

  /**
   * Current completion state of the onboarding task.
   */
  status: 'OPEN' | 'IN_PROGRESS' | 'DONE';

  /**
   * Called when the user opens the task detail view.
   */
  onClick: () => void;
};
```

Avoid documenting obvious props such as:

```tsx
type Props = {
  id: string;
  className?: string;
};
```

unless additional context is required.

---

## Functions and Business Logic

Functions should be documented whenever they contain business logic or behavior that is not immediately obvious.

### Async Operations

```tsx
/**
 * Loads the current user profile when the application starts.
 *
 * The result determines whether the user can access protected routes
 * or needs to complete the role selection first.
 */
const initAuth = async () => {
  // ...
};
```

### User Actions

```tsx
/**
 * Saves the selected working area in the backend.
 *
 * After a successful update, the user can access the generated onboarding path.
 */
const handleSubmitRoleSelection = async () => {
  // ...
};
```

### Complex Logic

Document:

* Business rules
* Conditional flows
* Data transformations
* Permission handling
* State synchronization

Do not document trivial code.

---

## useEffect Documentation

Simple effects do not require documentation.

Document effects when:

* They trigger backend communication.
* They synchronize state.
* They depend on multiple conditions.
* Their execution timing is important.

**Example**

```tsx
useEffect(() => {
  /**
   * Loads the onboarding path once the authenticated user profile is available.
   * The backend requires the user ID to return the correct path.
   */
  const loadPath = async () => {
    // ...
  };

  if (profile?.id) {
    void loadPath();
  }
}, [profile?.id]);
```

---

## Service Functions

All service functions responsible for backend communication should be documented.

Documentation should include:

* Purpose
* Parameters
* Return value (if necessary)
* Possible errors

**Example**

```tsx
/**
 * Fetches the onboarding path for a specific user.
 *
 * @param userId - Backend ID of the authenticated user.
 * @throws If the backend request fails.
 */
export async function fetchOnboardingPath(
  userId: string
): Promise<OnboardingPath> {
  // ...
}
```

---

## What Should Not Be Documented

Avoid documentation for:

* Obvious variable assignments
* Simple state updates
* Basic JSX markup
* Trivial helper functions
* Self-explanatory code

**Bad Example**

```tsx
// Set loading state
setLoading(true);

// Navigate to onboarding page
navigate('/onboarding');
```

These comments do not provide additional value.

---

## Documentation Checklist

Before creating a pull request, verify the following:

* [ ] Large page/view components are documented.
* [ ] Reusable components are documented when necessary.
* [ ] Complex props are documented.
* [ ] Business logic contains explanatory comments where needed.
* [ ] Non-trivial `useEffect` hooks are documented.
* [ ] Backend service functions are documented.
* [ ] Documentation explains intent and context, not obvious implementation details.
* [ ] Documentation reflects the current implementation.
