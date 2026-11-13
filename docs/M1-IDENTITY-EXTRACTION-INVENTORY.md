# M1 Identity extraction inventory and boundary gate

**Status:** inventory complete; source transfer intentionally not started  
**Source:** `andalus-ai/numu/identity/` on `main`  
**Source tree inventory:** 306 tracked files  
**Source reactor:** 3 active modules plus a legacy BOM POM  
**Related Core boundary:** Tenancy remains in `andalus-ai/numu` Core with Foundation  
**Related Security staging:** `andalus-ai/numou-security` PR #2 is staged, not yet accepted

## 1. Observed source inventory

The active source reactor in `identity/pom.xml` declares exactly these modules:

| Source module | Tracked files | Current contents |
|---|---:|---|
| `numou-identity-domain` | 55 | User aggregate, identity-provider links/configuration, token/client-credential models, role/permission models, tenant membership models, events and value objects |
| `numou-identity-application` | 129 | User use cases, federation, SCIM, OAuth client credentials, PAT/service accounts, authorization and ABAC services, tenant-selection and membership ports |
| `numou-identity-adapter` | 118 | Quarkus REST/gRPC, PostgreSQL repositories and 9 migrations, JWT/password adapters, OIDC/LDAP/SAML, Kafka outbox, health and tests |
| Root metadata and legacy BOM | 4 | Root POM, README, .gitignore and `bom/pom.xml` |
| **Total** | **306** | Full tracked tree under `identity/` |

The counts are an inventory of tracked paths, not a claim that every class is production-ready. Tests,
resources, SQL migrations, test keys/fixtures, protocol definitions and application configuration
must all be included in the eventual source-to-target manifest.

## 2. Ownership corrections required before source transfer

The current source reactor is named Identity, but its implementation is a mixed identity/security/
tenancy service. A directory name is not a valid ownership boundary. Do not copy all 306 files as
one accepted Identity capability.

### Identity-owned target

- Canonical identity record and lifecycle, with a stable subject identifier.
- Person and legal-organization identity, including explicit relationship/identifier contracts.
- Profile, contact point, address and external identity-link records where those concepts are
  canonical identity data rather than customer/workforce business profiles.
- Public application ports for resolving and maintaining canonical identity references.
- Identity lifecycle events and identity-owned persistence/migrations.

### Security-owned target

The following concerns must be moved behind Security contracts or extracted into Security-owned
modules before the Identity adapter is considered independently deployable:

- Password verification/hash lifecycle, login and authentication flows.
- OAuth token issuance/introspection/JWKS, client credentials and token validation.
- Personal access tokens, service-account authentication and credential lifecycle.
- Role/permission models, RBAC/ABAC evaluation, authorization services and policy persistence.
- Security enforcement filters/guards, authentication-specific REST resources and security audit.

The current Identity adapter directly depends on `numou-security-spi` and
`numou-security-application`. That dependency is a known boundary coupling, not a dependency to
silently preserve. Security must not depend back on Identity implementation classes. Define the
small stable subject-reference contract at the correct shared boundary and test the dependency
direction to prevent a cycle.

### Core Platform / Tenancy-owned target

- Tenant lifecycle, tenant membership, tenant discovery/provisioning and tenant isolation context
  belong to Core Tenancy.
- The current Identity domain includes `TenantMembership` and membership lifecycle events; the
  application includes tenant membership/selection ports and the adapter includes tenant membership
  persistence and discovery adapters.
- Identity may retain a reference to a tenant and a canonical subject, but it must not become a
  second owner of tenant membership state or lifecycle.

### Business capability-owned target

Customer, employee/worker, supplier, merchant and driver profiles are business relationships or
roles. They may reference canonical Identity subjects, but must not be folded into a universal
Identity profile aggregate.

## 3. Build and packaging findings

1. The source root inherits `numou-framework` through
   `../core/framework/pom.xml`. The destination must use the neutral published
   `numou-parent` and import the Core platform `numou-bom`; it must not rely on a monorepo
   relative path.
2. The source `identity/bom/pom.xml` incorrectly reuses artifactId `numou-bom`, which is the
   shared Core platform BOM identity. Do not copy it unchanged. The destination needs a unique
   `numou-identity-bom` covering only Identity-owned artifacts.
3. The source adapter declares direct Security dependencies and Core Foundation/Cache/Observability
   dependencies. The final dependency graph must be narrowed to the contracts actually used after
   the ownership split.
4. The adapter POM records Quarkus 3.15.1 and explicitly warns that it has not been compiled against
   the real Quarkus BOM. Treat the full adapter build as an unverified acceptance gate.
5. Preserve the 9 SQL migrations in order, their checksums/semantics, outbox behavior, gRPC proto,
   test resources and integration tests. Do not regenerate or reorder migrations during extraction.

## 4. Extraction sequence

| Step | Work | Exit criteria |
|---|---|---|
| M1.1 | Freeze source inventory and ownership map | Every tracked file classified as Identity, Security, Core Tenancy, business-owned, or deferred |
| M1.2 | Define stable subject and cross-capability contracts | No Identity-to-Security implementation dependency and no Security-to-Identity cycle |
| M1.3 | Separate mixed source modules | Identity reactor contains only canonical identity behavior; Security and Core retain their own invariants |
| M1.4 | Create destination Maven parent, unique BOM and module reactor | No monorepo relative paths; only declared public artifacts and explicit versions |
| M1.5 | Transfer Identity-owned code and migration history | Source-to-target manifest accounts for every included/excluded file |
| M1.6 | Verify clean-room build and integration | Domain/application tests, adapter integration tests, migrations, outbox and contract tests pass |
| M1.7 | Migrate consumers and deprecate source paths | No consumer relies on old implementation packages or duplicate persistence ownership |

## 5. Required source-to-target manifest

Before code transfer, add a machine-reviewable manifest with one row per tracked source path and
one disposition:

- `identity`: move to the Identity repository.
- `security`: remain/move under Security ownership.
- `core-tenancy`: remain under Core Tenancy.
- `business-capability`: move to the owning business capability in a later approved wave.
- `deferred`: retain in the monorepo pending an explicit decision.

The manifest must include file path, source blob SHA, disposition, target path (when applicable),
and rationale for every non-Identity disposition. No file is deleted from the source monorepo by
this inventory milestone.

## 6. Acceptance checklist

- [x] Read the source repository `AGENTS.md` and ADR-003 before changing the extraction plan.
- [x] Enumerate the complete tracked `identity/` tree and active Maven reactor.
- [x] Identify the direct Security dependency and Core Tenancy overlap.
- [x] Identify the conflicting legacy BOM artifactId.
- [ ] Approve the Identity/Security/Core subject-reference contract.
- [ ] Complete the per-file source-to-target manifest.
- [ ] Publish and resolve the Core parent/BOM/Foundation/Cache/Observability/Security contracts.
- [ ] Transfer only Identity-owned files.
- [ ] Pass a clean independent Maven build and relevant integration tests.
- [ ] Migrate consumers before deprecating monorepo source.

## 7. Verification statement

This PR changes destination documentation only. It does not transfer source code, change the source
monorepo, publish Maven artifacts, alter database schemas, or claim a successful Identity build.
