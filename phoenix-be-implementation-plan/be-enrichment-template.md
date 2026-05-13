# Backend implementation enrichment (Section 3)

> Replace or enrich the design-level Section 3 with code-audited detail from `phoenix`.

## 3.1 Summary
[Updated after codebase exploration]

## 3.2 Similar features / patterns to follow

> Prefer **user-attached reference files** when provided. Otherwise use in-repo equivalents.

| Reference feature | Controller | Service | Why relevant |
|-------------------|------------|---------|--------------|
| ... | `path/...` | `path/...` | ... |

## 3.3 Codebase anchors (verified)

| Layer | Path | Action |
|-------|------|--------|
| Controller | `registration/src/main/java/...` | create \| extend |
| Service | `...` | create \| extend |
| API types | `.../rest/types/...` | create \| extend |
| Repository / entity | `...` | create \| extend |
| Migration | `registration/src/main/resources/db/migration/V...__....sql` | create \| none |
| Controller test | `registration/src/test/java/...` | create \| extend |
| Service test | `registration/src/test/java/...` | create \| extend |

## 3.4 Data model & migrations

| Change | Type | Migration file | Notes |
|--------|------|----------------|-------|
| ... | alter \| new table \| index \| seed | `V...__....sql` | ... |

## 3.5 API contract adjustments (if any)

> Update Section 2 if exploration changed paths, auth, or payload shapes.

| Change | Reason |
|--------|--------|
| ... | ... |

## 3.6 Implementation steps (ordered)

| Step | Task | File(s) | Depends on | Done when |
|------|------|---------|------------|-----------|
| BE-1 | ... | `...` | — | ... |
| BE-2 | Flyway migration | `V...__....sql` | — | migration applies cleanly |
| BE-3 | Service logic | `...Service.java` | BE-2 | unit tests pass |
| BE-4 | API types | `Api....java` | — | types match contract |
| BE-5 | Controller endpoint | `...Controller.java` | BE-3, BE-4 | controller test passes |
| BE-6 | Tests | `...Test.java` | BE-5 | green locally |

### Step detail

#### BE-N: [Title]
- **What:**
- **Where:** `exact/path`
- **Pattern:** follow `[SimilarClass]`
- **Auth:** `@PreAuthorize(...)` — match `[reference endpoint]`
- **Errors:** `BusinessException` / `NotFoundException` — when
- **Done when:**

## 3.7 Verification checklist (BE)

- [ ] `./gradlew :registration:test --tests "...Test"` (or targeted equivalent)
- [ ] Migration runs on local DB
- [ ] Manual API check: `[METHOD] [path]` with role `[role]`
- [ ] Section 2 contract matches implemented shapes
