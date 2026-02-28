# Removal and Iteration Plan Template

## Priority Levels

- [ ] **P0**: Immediate removal needed (security risk, significant cost, blocking other work)
- [ ] **P1**: Remove in current sprint
- [ ] **P2**: Backlog / next iteration
- [ ] **P3**: Deprecation Notice Added (Code is active, but warnings are emitted to consumers. Start of the sunset clock).

---

## Safe to Remove Now

### Item: [Name/Description]

| Field | Details |
|-------|---------|
| **Location** | `path/to/file.ts:line` |
| **Rationale** | Why this should be removed |
| **Evidence** | Unused (no references), dead feature flag, deprecated API |
| **Impact** | None / Low - no active consumers |
| **Deletion steps** | 1. Remove code 2. Remove tests 3. Remove config |
| **Verification** | Run tests, check no runtime errors, monitor logs |
| **Database Impact** | Are there orphaned tables/columns that need a separate migration plan? |
| **Dependency Cleanup** | Should `package.json` / `go.mod` / `pom.xml` dependencies be removed because of this code deletion? |

---

## Defer Removal (Plan Required)

### Item: [Name/Description]

| Field | Details |
|-------|---------|
| **Location** | `path/to/file.ts:line` |
| **Why defer** | Active consumers, needs migration, stakeholder sign-off |
| **Preconditions** | Feature flag off for 2 weeks, telemetry shows 0 usage |
| **Breaking changes** | List any API/contract changes |
| **Migration plan** | Steps for consumers to migrate |
| **Timeline** | Target date or sprint |
| **Owner** | Person/team responsible |
| **Validation** | Metrics to confirm safe removal (error rates, usage counts) |
| **Rollback plan** | How to restore if issues found |
| **Deprecation Communication** | How are consumers notified? (e.g., API response headers, changelog, direct email) |
| **Shadow Run / Dry Run Phase** | Can we log when the old code *would* have been hit, without actually executing it, to verify 0% usage? |

---

## Checklist Before Removal

- [ ] Searched codebase for all references (`rg`, `grep`)
- [ ] Checked for dynamic/reflection-based usage
- [ ] Verified no external consumers (APIs, SDKs, docs)
- [ ] Feature flag telemetry reviewed (if applicable)
- [ ] Tests updated/removed
- [ ] Documentation updated
- [ ] Team notified (if shared code)
- [ ] Database migrations (Drop tables/columns) are sequenced correctly (usually after the code is fully deployed).
- [ ] Removed stale dependencies from dependency manager (e.g., npm, pip, maven).
- [ ] Removed related environment variables, secrets, or configuration flags from CI/CD and deployment manifests.
- [ ] Checked for implicitly related assets (e.g., old images, translation strings, cron jobs).
