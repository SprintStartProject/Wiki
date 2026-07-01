# ADR-015: Use Keycloak for Authentication and Authorization

| Field | Value |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-06-07 |
| **Deciders** | Backend Team |
| **Supersedes** | – |
| **Superseded by** | – |

---

## Context

SprintStart requires authentication and authorization for users. Different users need different access levels depending on their role in the system, for example regular users or administrators.

The application should not implement its own password handling, login flow, token issuing, or user session management. These areas are security-critical and difficult to implement correctly. Instead, the system should rely on a dedicated identity provider.

The backend still needs local application-specific user data, such as onboarding-related data, and references to onboarding paths. However, authentication data such as passwords, login credentials, and access roles should be managed outside the application database.

## Decision Drivers

- Avoid implementing custom authentication and password handling.
- Support role-based access control for different user access levels.
- Use a standardized authentication and authorization mechanism.
- Keep authentication concerns separate from application-specific user data.
- Integrate cleanly with the Spring Boot backend.
- Support future extensions such as groups, external identity providers, email verification, and multi-factor authentication.

## Considered Options

- **Option A** – Implement authentication and authorization manually in the backend.
- **Option B** – Use OAuth 2.0 / OpenID Connect directly without a dedicated identity provider.
- **Option C** – Use Keycloak as the identity provider.

## Decision

**Chosen option: Option C** – Use Keycloak as the identity provider because it provides a complete authentication and authorization solution based on standard protocols and integrates well with Spring Security.

## Rationale

Keycloak provides user management, login handling, token issuing, role management, groups, and integration with standard protocols such as OAuth 2.0 and OpenID Connect.

OAuth 2.0 and OpenID Connect are protocols, not complete user management systems. Using them directly would still require the team to provide or integrate an identity provider. Keycloak gives the project a ready-to-use identity provider while still relying on established standards.

The SprintStart backend will validate access tokens issued by Keycloak. Authorization decisions in the backend can then be based on roles or claims contained in the token. Application-specific user data will remain in the SprintStart database and will be linked to the Keycloak user through the Keycloak user ID.

## Pros and Cons of the Options

### Option A – Manual authentication and authorization in the backend

- ✅ Full control over the authentication and authorization implementation.
- ✅ No additional infrastructure component required.
- ❌ High security risk due to custom password handling and token management.
- ❌ Requires implementation of login, password reset, credential storage, role handling, and session/token logic.
- ❌ More maintenance effort for the backend team.
- ❌ Harder to extend with external login providers or multi-factor authentication.

### Option B – Use OAuth 2.0 / OpenID Connect directly without a dedicated identity provider

- ✅ Uses standardized protocols.
- ✅ Avoids inventing a custom token format.
- ❌ OAuth 2.0 / OpenID Connect are protocols, not complete user management systems.
- ❌ Still requires an authorization server or identity provider.
- ❌ User management, role management, login UI, and password flows would still need to be provided separately.

### Option C – Use Keycloak as the identity provider

- ✅ Provides authentication, authorization, user management, roles, groups, and token issuing.
- ✅ Supports OAuth 2.0 and OpenID Connect.
- ✅ Integrates well with Spring Security.
- ✅ Avoids storing passwords in the SprintStart backend database.
- ✅ Allows access levels to be managed centrally through Keycloak roles.
- ✅ Can be extended later with external identity providers, email verification, and multi-factor authentication.
- ❌ Adds another infrastructure component that must be configured, deployed, and maintained.
- ❌ Requires local development setup for Keycloak.
- ❌ Requires mapping Keycloak roles/claims to Spring Security authorities.

## Consequences

**Positive:**
- Authentication is handled by a dedicated identity provider.
- The backend does not store or manage user passwords.
- Access levels can be represented through Keycloak roles.
- The backend can use Spring Security to validate JWT access tokens.
- The application keeps a clear separation between identity data and application-specific user data.

**Negative / Trade-offs:**
- Keycloak must be added to the local and deployment infrastructure.
- Developers need to understand the basic concepts of realms, clients, roles, users, and tokens.
- Token role mapping must be configured correctly in the backend.
- The backend still needs a local user/profile entity for application-specific data.

**Follow-up actions:**
- Add Keycloak to the local Docker Compose setup.
- Create a `sprintstart` realm.
- Create a frontend/client configuration for SprintStart.
- Define the required roles, for example `USER` and `ADMIN`.
- Configure Spring Security as an OAuth 2.0 Resource Server.
- Map Keycloak roles from the JWT to Spring Security authorities.
- Update the local `User` entity so it stores the Keycloak user ID instead of authentication credentials.
- Decide which user attributes are stored in Keycloak and which are stored in the SprintStart database.
- Document the local setup steps for developers.

## Links

- [ADR Index](index.md)
