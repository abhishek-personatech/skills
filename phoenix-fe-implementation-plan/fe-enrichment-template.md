# Frontend implementation enrichment (Section 4)

> Replace or enrich the design-level Section 4 with code-audited detail from `phoenix-fe`.

## 4.1 Summary
[Updated after codebase exploration]

## 4.2 Similar features / patterns to follow

> Prefer **user-attached reference files** when provided. Otherwise use in-repo equivalents.

| Reference feature | Path | Why relevant |
|-------------------|------|--------------|
| ... | `src/apps/...` | service hook + route + API pattern |

## 4.3 Codebase anchors (verified)

| Area | Path | Action |
|------|------|--------|
| App / surface | `src/apps/console/` \| `src/apps/myExperience/` | ... |
| Feature root | `src/apps/.../modules/.../FeatureName/` | create \| extend |
| Route | `.../routes/...` | add route \| extend config |
| API enum | `consoleApi.ts` \| `MyExpApi.ts` | add constant(s) |
| Data loader / API | `.../dataLoader.ts` or `service.ts` | add function(s) |
| Types | `types.ts` | create \| extend |
| i18n keys | `I18n` — `[key.group.*]` | add keys |
| Tests | `__test__/FeatureName.test.tsx` | create |
| SCSS | `FeatureName.scss` | create \| none |

## 4.4 Proposed feature structure

```
src/apps/[app]/.../FeatureName/
├── index.tsx
├── service.ts
├── types.ts
├── components/
│   └── ...
├── FeatureName.scss          # if needed
└── __test__/
    ├── FeatureName.test.tsx
    └── mockData.ts
```

## 4.5 API integration (verified against Section 2)

| UI action | API (Section 2) | API enum / URL builder | Service function | Request mapping | Response mapping | Query key / cache | Error UX |
|-----------|-----------------|------------------------|------------------|-----------------|------------------|-------------------|----------|
| ... | `GET /...` | `ConsoleApi....` | `fetch...` | ... | `types.ts` types | `['...']` | toast / inline |

**Integration notes**

- **HTTP client pattern:** [from similar feature — React Query hook in service.ts, etc.]
- **Auth / headers:** ...
- **Feature flag / config:** ...
- **Mock strategy for tests:** `mockGet`/`mockPost` from `tests/common/setupAxiosMock`

## 4.6 UX / copy (concrete)

| State | UI behavior | Copy keys |
|-------|-------------|-----------|
| Loading | ... | `...` |
| Success | ... | `...` |
| Empty | ... | `...` |
| Error | ... | `...` |

## 4.7 Implementation steps (ordered)

| Step | Task | File(s) | Depends on | Done when |
|------|------|---------|------------|-----------|
| FE-1 | Types | `types.ts` | Section 2 contract | compiles |
| FE-2 | API enum + client | `consoleApi.ts`, `service.ts` | FE-1 | mockable |
| FE-3 | Service hook | `service.ts` | FE-2 | hook testable |
| FE-4 | Components | `components/` | FE-3 | render states |
| FE-5 | Wire index + route | `index.tsx`, routes | FE-4 | navigable |
| FE-6 | i18n keys | `I18n` | FE-4 | no hardcoded strings |
| FE-7 | Integration tests | `__test__/` | FE-5 | per testing skill |
| FE-8 | SCSS | `.scss` | FE-4 | per phoenix-scss |

### Step detail

#### FE-N: [Title]
- **What:**
- **Where:** `exact/path`
- **Pattern:** follow `[SimilarFeature]`
- **Done when:**

## 4.8 Verification checklist (FE)

- [ ] Targeted test: `npm test -- --testPathPattern=FeatureName` (or repo-standard command)
- [ ] Manual check on [console \| my-experience]: route, happy path, error path
- [ ] API integration matches Section 2 (and BE enriched contract if present)
- [ ] `CLAUDE.md` + testing skill conventions satisfied
