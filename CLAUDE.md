# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **archiva-redback-core** (9105 symbols, 33098 relationships, 300 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `gitnexus_context({name: "symbolName"})`.

## Never Do

- NEVER edit a function, class, or method without first running `gitnexus_impact` on it.
- NEVER ignore HIGH or CRITICAL warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `gitnexus_rename` which understands the call graph.
- NEVER commit changes without running `gitnexus_detect_changes()` to check affected scope.

## Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/archiva-redback-core/context` | Codebase overview, check index freshness |
| `gitnexus://repo/archiva-redback-core/clusters` | All functional areas |
| `gitnexus://repo/archiva-redback-core/processes` | All execution flows |
| `gitnexus://repo/archiva-redback-core/process/{name}` | Step-by-step execution trace |

## CLI

| Task | Read this skill file |
|------|---------------------|
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus/gitnexus-cli/SKILL.md` |

<!-- gitnexus:end -->

# Maven Build & Test

This is a Maven multi-module project (v3.0.0-SNAPSHOT, JDK 11+). Parent POM is `org.apache.archiva:redback:3.0.0-SNAPSHOT`.

## Common Commands

```bash
# Full build (compile + test + package)
mvn clean package

# Build without tests
mvn clean package -DskipTests

# Run a single test class
mvn test -Dtest=FullyQualifiedTestClassName

# Run tests in a specific module
mvn test -pl redback-users

# Run tests matching a pattern in all modules
mvn test -Dtest="*ManagerTest"

# Build site documentation
mvn clean site site:stage

# RAT license header check (runs in verify phase)
mvn verify

# Release preparation
mvn release:prepare
```

## Test Configuration

- Tests use JUnit 5 (Jupiter) with Vintage engine for JUnit 4 compatibility.
- Test database: HSQLDB in-memory (`jdbc:hsqldb:mem:redback-test`).
- Surefire runs tests in alphabetical order with `-Xmx256m -Xms256m`.
- 73 test files across the project.

## Key Dependencies

- **Spring Framework 5.3.x** — IoC, AOP, transaction management
- **Apache CXF 3.3.x** — JAX-RS REST API framework
- **Apache OpenJPA 3.1.x** — JPA persistence
- **Jackson 2.12.x** — JSON/XML serialization
- **Jakarta EE** — Servlet, Persistence, Transaction, WS-RS APIs
- **Ehcache 3.9.x** — Caching layer
- **JJWT 0.11.x** — JWT token handling

# Architecture

Redback is an authorization and authentication framework used by Apache Archiva. It follows a layered, provider-based architecture where core APIs are implemented by pluggable backends (JPA, LDAP, memory, cached, configurable).

## Module Structure

```
redback/                              # Parent POM
├── redback-authentication/           # Authentication layer
│   ├── redback-authentication-api/   # Core interfaces: AuthenticationManager, Authenticator, Token, TokenData
│   └── redback-authentication-providers/
│       ├── redback-authentication-jwt/       # JWT-based authentication
│       ├── redback-authentication-ldap/      # LDAP authentication
│       ├── redback-authentication-memory/    # In-memory authentication
│       ├── redback-authentication-open/      # Open/unauthenticated mode
│       └── redback-authentication-users/     # User-based authentication
├── redback-authorization/            # Authorization layer
│   ├── redback-authorization-api/    # Core interface: Authorizer (isAuthorized → AuthorizationResult)
│   └── redback-authorization-providers/
├── redback-rbac/                     # RBAC engine
│   ├── redback-rbac-model/           # Domain model: Role, Permission, Operation, Resource, UserAssignment
│   ├── redback-rbac-providers/       # RBAC backends (memory, JPA, cached)
│   ├── redback-rbac-role-manager/    # Role management: RoleManager
│   └── redback-rbac-tests/           # Integration tests
├── redback-users/                    # User management
│   ├── redback-users-api/            # Core interface: UserManager, User, UserQuery
│   ├── redback-users-providers/      # User backends (JPA, LDAP, memory, cached, configurable)
│   └── redback-users-tests/          # Integration tests
├── redback-keys/                     # Authentication key management
│   ├── redback-keys-api/             # Core interface: KeyManager, AuthenticationKey
│   ├── redback-keys-providers/       # Key backends (JPA, memory, cached)
│   ├── redback-authentication-keys/  # Key-based authentication
│   └── redback-keys-tests/           # Integration tests
├── redback-common/                   # Shared utilities
│   ├── redback-common-jpa/           # JPA base classes, entity helpers
│   ├── redback-common-ldap/          # LDAP utilities
│   ├── redback-common-configuration/ # Configuration model (UserConfiguration)
│   └── redback-common-test-resources/ # Test fixtures
├── redback-system/                   # Unified security facade
│   # SecuritySystem — aggregates authentication, authorization, user management, key management, and policy
├── redback-configuration/            # Configuration model
├── redback-policy/                   # Security policies (password rules, cookie settings, user validation)
├── redback-integrations/             # Integration layers
│   └── redback-rest/                 # REST API layer
│       ├── redback-rest-api/         # JAX-RS service interfaces (v1 deprecated, v2 current)
│       └── redback-rest-services/    # JAX-RS service implementations
└── redback-policy/                   # Security policies (password rules, cookie settings)
```

## Core Interfaces

| Interface | Module | Purpose |
|-----------|--------|---------|
| `SecuritySystem` | redback-system | Unified facade — aggregates authentication, authorization, user management, key management, and policy |
| `AuthenticationManager` | redback-authentication-api | Orchestrates authentication via ordered `Authenticator` chain |
| `Authenticator` | redback-authentication-api | Single authentication source (LDAP, JWT, memory, etc.) |
| `UserManager` | redback-users-api | CRUD operations on users, queries, guest user support |
| `RBACManager` | redback-rbac-model | Role/permission/operation/resource management, user-role assignments |
| `Authorizer` | redback-authorization-api | Authorization checks returning `AuthorizationResult` |
| `KeyManager` | redback-keys-api | Authentication key lifecycle (create, validate, expire) |
| `RoleManager` | redback-rbac-role-manager | Higher-level role management operations |

## Provider Pattern

Each layer (authentication, authorization, users, RBAC, keys) follows a **provider pattern**:

1. **API module** — defines the core interface (e.g., `UserManager`)
2. **Providers module** — contains implementations:
   - `*-jpa` — JPA-backed persistence via OpenJPA
   - `*-ldap` — LDAP-backed storage
   - `*-memory` — in-memory (for testing/embedded use)
   - `*-cached` — Ehcache-backed wrapper around another provider
   - `*-configurable` — runtime-swappable wrapper

Implementations declare `isFinalImplementation()`, `getDescriptionKey()`, and `initialize()` for lifecycle management.

## REST API

The REST API (`redback-rest`) uses JAX-RS (Jakarta WS-RS) with Apache CXF. Two versions exist:

- **v1** (deprecated): `LoginService`, `UserService`, `PasswordService`, `RoleManagementService`, `LdapGroupMappingService`
- **v2** (current): `AuthenticationService`, `UserService`, `RoleService`, `GroupService`

Authorization on REST endpoints is controlled via the `@RedbackAuthorization` annotation.

## Data Flow

1. **Authentication**: Client → REST API → `SecuritySystem.authenticate()` → `AuthenticationManager` → chain of `Authenticator` → `UserManager` → returns `SecuritySession`/`Token`
2. **Authorization**: `SecuritySystem.authorize()` → `Authorizer` chain → `RBACManager` (resolves user's roles → permissions)
3. **User Management**: REST API → `SecuritySystem.getUserManager()` → specific `UserManager` implementation (JPA/LDAP/memory)
4. **RBAC**: `RBACManager` operations on `Role`/`Permission`/`Operation`/`Resource`/`UserAssignment` entities, persisted via JPA or cached

## Security Model

- **Role** contains **Permissions** and child **Roles** (hierarchical)
- **Permission** = Operation + Resource combination
- **UserAssignment** maps users to roles
- `RBACManager.getEffectiveRoles()` resolves the full role hierarchy for a user
- `RBACManager.getAssignedPermissions()` resolves all permissions from all effective roles
