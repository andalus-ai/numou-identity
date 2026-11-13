# Numou Identity

Identity is the canonical identity capability for the Numou platform.

**Migration status: inventory and boundary review.** The authoritative implementation remains in
`andalus-ai/numu/identity/` until the extraction is independently buildable, its contracts are
reviewed, and consumers have been migrated. This repository is not yet a release source.

## Ownership

Identity is intended to own canonical person and organization identity, stable subject references,
identity lifecycle, profile/contact/address records, and external identity links.

It does **not** own:
- Authentication, credential verification, MFA, sessions, authorization policy or security audit (Security).
- Tenant lifecycle, tenant membership, provisioning or tenant isolation context (Core Platform / Tenancy).
- Customer, employee, supplier, merchant or driver lifecycle (their respective business capabilities).

See [the extraction inventory and boundary gate](docs/M1-IDENTITY-EXTRACTION-INVENTORY.md) before
adding or moving implementation code.

## Current source of truth

- Source repository: [andalus-ai/numu](https://github.com/andalus-ai/numu)
- Source reactor: `identity/pom.xml`
- Active source modules: domain, application, adapter
- Migration acceptance requires a clean independent Maven build, contract and data compatibility
  checks, adapter/integration tests, and a consumer migration plan.

No source files have been removed from the monorepo.
