# Data Classification — 5 PR Split Plan

Source: [PR #8726](https://github.com/upwindsecurity/frontend-monorepo/pull/8726) (75 files)

> ⚠️ **Foundation review moved capabilities cell path.** During PR1 review, `ClassificationCapabilitiesCell.tsx` + `ClassificationCapabilitiesInteractive.tsx` were moved out of `components/DataClassifications/ClassificationCapabilitiesInteractive/` and into `pages/DataPage/components/DataClassificationsTab/ClassificationsTable/components/` (co-located with the other table cells).
>
> Affected feature PRs:
> - **PR2**: drop the `D pages/.../ClassificationCapabilitiesCell.tsx` line — foundation now keeps the file at this path (with new content)
> - **PR4** (`ClassificationPatternsTableColumnDefs.ts`) and **PR5** (`GlobalExclusionsTableColumnDefs.ts`): update the `ClassificationCapabilitiesCell` import to the new co-located path
>
> ⚠️ **Foundation review promoted DisableItemModal to shared-library as ConfirmActionModal.** Generic confirmation modal with required `cancelText` / `confirmText` and `confirmButtonType?: "confirm" | "delete"` (defaults to `"delete"`).
>
> Affected feature PRs (5 callers, all currently using `<DisableItemModal>`):
> - **PR4**: `ClassificationDetailsOverviewTab.tsx`, `DeletePatternModal.tsx`, `EditPatternModal.tsx`, `ClassificationDetailsSidePane.tsx`
> - **PR5**: `GlobalExclusionsSidePane.tsx`
>
> Each caller needs:
> - Replace `import { DisableItemModal } from ".../DisableItemModal"` with `import { ConfirmActionModal } from "@frontend-monorepo/shared-library"`
> - Rename: `<DisableItemModal>` → `<ConfirmActionModal>`, `onConfirmDisable` → `onConfirm`, `confirmQuestion` → `question`, `confirmDescription` → `description`
> - **Pass explicit `cancelText` and `confirmText` props** (no defaults anymore) — the three callers that previously relied on `"No, keep policy"` / `"Yes, I'm sure"` defaults need correct domain-specific text
>
> ⚠️ **Foundation review converted `VALIDATION_STATE` to `ValidationState` enum and relocated it.** Was `export const VALIDATION_STATE = {...} as const;` + derived type in `PatternEntryRow.tsx`. Now `export enum ValidationState` in `DataModel/DataClassificationTypes/ClassificationTypes.ts`.
>
> Affected feature PRs:
> - **PR3**: `ClassificationPatternsSection.tsx`
> - **PR4**: `AddPatternModal.tsx`, `EditPatternModal.tsx`, `usePatternValidator.ts`
>
> Each caller needs:
> - Change import from `.../PatternEntryRow` to `.../DataModel/DataClassificationTypes/ClassificationTypes`
> - Rename `VALIDATION_STATE` → `ValidationState` (member access `.IDLE` / `.ERROR` / `.LOADING` / `.VALIDATED` unchanged — enum string values match the old object values)
>
> ⚠️ **Foundation review converted `CLASSIFICATION_SOURCE` to `ClassificationSource` enum.** Was `export const CLASSIFICATION_SOURCE = {...} as const;` + derived type. Now `export enum ClassificationSource` in `ClassificationTypes.ts`. (`ClassificationSource` type name unchanged.)
>
> Affected feature PR:
> - **PR4**: `PatternActionsCell.tsx` uses `CLASSIFICATION_SOURCE.CORE` — replace with `ClassificationSource.CORE` (and update import to drop `CLASSIFICATION_SOURCE`, keep/add `ClassificationSource`).
>
> ⚠️ **Foundation review removed `ClassificationFindingRiskOverview` alias re-export.** Was `export type { FindingRiskOverview as ClassificationFindingRiskOverview };` in `ServerAPI/.../dataClassificationTypes.ts`. Removed — callers should import the original `FindingRiskOverview` directly from `DataModel/NewPostureDataModel/Types/ConfigurationsFindingsTypes`.
>
> Affected feature PR:
> - **PR4**: `ClassificationFindingsSection.tsx` imports `ClassificationFindingRiskOverview` — replace import with `FindingRiskOverview` from `DataModel/NewPostureDataModel/Types/ConfigurationsFindingsTypes`, and rename the local usages.
>
> ⚠️ **PR4 absorbs `TestPatternSection.tsx` (originally PR3).** Three callers use the new `externalPattern` prop on `<TestPatternSection>`: `ClassificationPatternsSection.tsx` (PR3), `AddPatternModal.tsx` and `EditPatternModal.tsx` (both PR4). The prop is added in `TestPatternSection.tsx`, so any PR using the prop depends on the file change. To make PR4 self-contained, `TestPatternSection.tsx` + `test-pattern-section.scss` are moved from PR3's file list to PR4's.
>
> **Side effect — merge order constraint:** PR3 still uses `<TestPatternSection externalPattern={…}>` in `ClassificationPatternsSection.tsx`, so **PR3 must merge after PR4**. PR2 and PR5 remain order-independent.
>
> ⚠️ **PR4 touches one line in `ClassificationsFilterConfig.ts` (PR2-owned file).** Adding `PENDING` to the `ClassificationStatus` enum (PR4) forces `Record<ClassificationStatus, …>` consumers to add a `PENDING` entry. `CLASSIFICATION_STATUS_TRANSLATION` in `ClassificationsFilterConfig.ts` is one such consumer. **PR4 adds the `PENDING: "Under evaluation"` line.** PR2 does not need to add it — when PR2 rebases on top of merged PR4, 3-way merge picks up the existing line cleanly.
>
> ⚠️ **`hasDuplicateScopingFilters` helper duplicated in PR3 and PR4 for order-independence.** Both `CreateDataClassificationSidePane.tsx` (PR3) and `ClassificationDetailsSettingsTab.tsx` (PR4) define an identical local `hasDuplicateScopingFilters` helper (uses `lodash.uniqWith` + `xorWith`) and call it in their submit handlers to toast and abort on duplicate scoping filters. **When PR3 merges, fold the dedup: extract the helper into `pages/.../DataClassificationScopeSelection/scopeSelectionUtils.ts` as a shared export, and replace both local copies with the import in the same PR3 commit.** No separate follow-up PR needed.

**Totals:** Foundation 24 + standalone deletion 1 + PR2 9 + PR3 8 + PR4 26 + PR5 9 = 77 (PR4 shares two files with other PRs: `ClassificationsFilterConfig.ts` is also in PR2's 9, and `ClassificationTypes.ts` is also in PR1's 24 → 75 unique files)

**Legend:** `M` modified · `A` added · `D` deleted

**Path convention:**

- Default: relative to `packages/console/src/`
- `sidepanes:` prefix: relative to `packages/console/src/components/GeneralSidePane/SidePanes/DataPage/DataClassifications/`
- `shared-library:` prefix: relative to `packages/shared-library/src/`

**Dependency graph:** `PR2, PR3, PR4, PR5 → PR1`, plus `PR3 → PR4` (PR3 uses `<TestPatternSection externalPattern={…}>` whose prop is added by PR4 — see migration note above). PR2 and PR5 stay order-independent; PR3 must merge after PR4.

---

## PR 1 · Foundation (24)

> Shared types, API surface, shared UI components and hooks.
> With feature flag `console-data-classifications` off, Foundation has zero behavioral impact on `main`.

### DataModel (5)

- M `DataModel/AiChat/AiSummaryPromptTypes.ts`
- M `DataModel/DataClassificationTypes/ClassificationScopeFilterTypes.ts`
- M `DataModel/DataClassificationTypes/ClassificationTypes.ts`
- M `DataModel/Filters/Views/ViewsTypes.ts`
- M `DataModel/Objects/DataClassification/Classification.ts`

### ServerAPI (2)

- M `ServerAPI/Requests/DataClassification/DataClassification.ts`
- M `ServerAPI/Requests/DataClassification/dataClassificationTypes.ts`

### Shared Components / Hooks (6)

- M `components/ConsoleFilterBar/FilterIconsConfiguration.tsx`
- A `pages/DataPage/components/DataClassificationsTab/ClassificationsTable/components/ClassificationCapabilitiesInteractive.tsx`
- M `pages/DataPage/components/DataClassificationsTab/ClassificationsTable/components/ClassificationCapabilitiesCell.tsx`
- A `shared-library: components/ConfirmActionModal/ConfirmActionModal.tsx`
- M `sidepanes: CreateDataClassificationSidePane/ClassificationPatternsSection/PatternEntryRow.tsx`
- A `sidepanes: hooks/useAiRegexGenerator.ts`

### ReporterSource Shared (2)

- A `components/ReporterSource/ReporterSourceBadgeIcon.tsx`
- M `components/Tables/VulnerabilitiesTables/NestedTables/asyncCellComponents/SourceCell/SourceCell.tsx`

### Scope / Capabilities Shared (6)

- M `pages/DataPage/components/DataClassificationsTab/components/DataClassificationScopeSelection/DataClassificationScopeViewSection.tsx`
- M `pages/DataPage/components/DataClassificationsTab/components/DataClassificationScopeSelection/ScopeViewConditionRow.tsx`
- A `pages/DataPage/components/DataClassificationsTab/components/DataClassificationScopeSelection/hooks/useFileTypeOptions.tsx`
- M `pages/DataPage/components/DataClassificationsTab/components/DataClassificationScopeSelection/hooks/useFilterValueOptions.ts`
- M `pages/DataPage/components/DataClassificationsTab/components/DataClassificationScopeSelection/scopeSelectionUtils.ts`
- M `pages/DataPage/components/DataClassificationsTab/components/SelectClassificationCapabilities/SelectClassificationCapabilities.tsx`

### shared-library (3)

- M `shared-library: DataModelCore/Filters/filterTypesDictionary.ts`
- M `shared-library: components/InputSearchAi/InputSearchAi.tsx`
- M `shared-library: components/TableCells/DefaultBadgeCodeCell/DefaultBadgeCodeCell.tsx`

### Backward-compat adjustments in Foundation

To keep Foundation strictly additive (compiles cleanly on `main` with the feature flag off), three originally-breaking changes were softened or deferred:

| Change | Foundation form | Restored / consumed by |
| --- | --- | --- |
| `PatternEntryRow.validationState`, `onBlur` props | Made optional with defaults (`IDLE`, no-op) | PR3 (`ClassificationPatternsSection`) passes them. Leave optional permanently — no restoration needed. |
| `CreatePatternRequestBody.scanTypes` field | Made optional via `Partial<Pick<…>>` | PR3 (`CreateDataClassificationSidePane`) and PR4 (Add/Edit pattern modals) pass it. Leave optional permanently. |
| `CLASSIFICATION_STATUS.PENDING` enum value + `CLASSIFICATION_STATUS_CONFIG[PENDING]` entry | **Removed from Foundation** | **PR4** re-adds both. Consumers: `ClassificationPatternsFilterConfig.ts` and `EditPatternModal.tsx` (lines 161/165/169). |

---

## Standalone commit (1)

- D `ServerAPI/Requests/DataClassification/DataClassificationApiStructure.md`

---

## PR 2 · Main Table (9)

> Data page entry point, main classifications table, column definitions and tab shell.

- M `DataModel/DataPageModel/ClassificationsFilterConfig.ts`
- M `components/UpwindPageLayout/UpwindPageLayout.tsx`
- M `pages/DataPage/DataPage.tsx`
- M `pages/DataPage/components/DataClassificationsTab/DataClassificationsTab.tsx`
- M `pages/DataPage/components/DataClassificationsTab/ClassificationsTable/ClassificationsTableColumnDefs.ts`
- M `pages/DataPage/components/DataClassificationsTab/ClassificationsTable/components/ClassificationCategoryCell.tsx`
- D `pages/DataPage/components/DataClassificationsTab/ClassificationsTable/components/ClassificationCapabilitiesCell.tsx`
- M `pages/DataPage/components/DataPageHeader/DataPageHeader.tsx`
- M `pages/IdentitiesPage/IdentitiesTabs/MachinesTab/MachineInnerTabs/ExposedSecretsGeneralPage/ExposedSecrets.tsx`

---

## PR 3 · Create Sidepane (8)

> Create-classification side pane and pattern testing.

- M `sidepanes: CreateDataClassificationSidePane/ClassificationPatternsSection/CapabilitiesOverrideSection.tsx`
- M `sidepanes: CreateDataClassificationSidePane/ClassificationPatternsSection/ClassificationPatternsSection.tsx`
- M `sidepanes: CreateDataClassificationSidePane/ClassificationPatternsSection/ExistingClassificationBadges.tsx`
- M `sidepanes: CreateDataClassificationSidePane/CreateClassificationBanner/CreateClassificationBanner.tsx`
- A `sidepanes: CreateDataClassificationSidePane/CreateClassificationBanner/createDataClassification-dark.png`
- M `sidepanes: CreateDataClassificationSidePane/CreateDataClassificationSidePane.tsx`
- M `sidepanes: CreateDataClassificationSidePane/SelectClassificationCapabilitiesSection/SelectClassificationCapabilitiesSection.tsx`
- M `sidepanes: CreateDataClassificationSidePane/SelectClassificationCategory/SelectClassificationCategory.tsx`

> ~~`TestPatternSection.tsx` + `test-pattern-section.scss`~~ moved to PR4 (see migration note above).

---

## PR 4 · Detail Sidepane (26)

> Classification detail side pane, Patterns tab, Findings section.

### Filter configs (3)

- A `DataModel/DataPageModel/ClassificationFindingsFilterConfig.ts`
- A `DataModel/DataPageModel/ClassificationFindingsFilterIconConfig.tsx`
- A `DataModel/DataPageModel/ClassificationPatternsFilterConfig.ts`

### Sidepane shell (4)

- M `sidepanes: ClassificationDetailsSidePane/ClassificationDetailsOverviewTab.tsx`
- M `sidepanes: ClassificationDetailsSidePane/ClassificationDetailsSettingsTab.tsx`
- M `sidepanes: ClassificationDetailsSidePane/ClassificationDetailsSidePane.tsx`
- D `sidepanes: ClassificationDetailsSidePane/DisableClassificationModal.tsx`

### Patterns tab (7)

- A `sidepanes: ClassificationDetailsSidePane/ClassificationDetailsPatternsTab/ClassificationDetailsPatternsTab.tsx`
- A `sidepanes: ClassificationDetailsSidePane/ClassificationDetailsPatternsTab/ClassificationPatternsTable/ClassificationPatternsTable.tsx`
- A `sidepanes: ClassificationDetailsSidePane/ClassificationDetailsPatternsTab/ClassificationPatternsTable/ClassificationPatternsTableColumnDefs.ts`
- A `sidepanes: ClassificationDetailsSidePane/ClassificationDetailsPatternsTab/ClassificationPatternsTable/PatternActionsCell.tsx`
- A `sidepanes: ClassificationDetailsSidePane/ClassificationDetailsPatternsTab/EditPatternModal/AddPatternModal.tsx`
- A `sidepanes: ClassificationDetailsSidePane/ClassificationDetailsPatternsTab/EditPatternModal/DeletePatternModal.tsx`
- A `sidepanes: ClassificationDetailsSidePane/ClassificationDetailsPatternsTab/EditPatternModal/EditPatternModal.tsx`

### Findings section (7)

- A `sidepanes: ClassificationDetailsSidePane/ClassificationFindingsSection/ClassificationFindingsSection.tsx`
- A `sidepanes: ClassificationDetailsSidePane/ClassificationFindingsSection/ClassificationFindingsTable/ClassificationFindingsTable.tsx`
- A `sidepanes: ClassificationDetailsSidePane/ClassificationFindingsSection/ClassificationFindingsTable/ClassificationFindingsTableColumnDefs.ts`
- A `sidepanes: ClassificationDetailsSidePane/ClassificationFindingsSection/ClassificationFindingsTable/cells/ClassificationFindingDataStateCell.tsx`
- A `sidepanes: ClassificationDetailsSidePane/ClassificationFindingsSection/ClassificationFindingsTable/cells/ClassificationFindingLocationCell.tsx`
- A `sidepanes: ClassificationDetailsSidePane/ClassificationFindingsSection/ClassificationFindingsTable/cells/ClassificationFindingSecurityIssuesCell.tsx`
- A `sidepanes: ClassificationDetailsSidePane/ClassificationFindingsSection/ClassificationFindingsTable/cells/ClassificationFindingSourceCell.tsx`

### Validator hook (1)

- A `sidepanes: hooks/usePatternValidator.ts`

### Test pattern section (2) — absorbed from PR3

- M `pages/DataPage/components/DataClassificationsTab/components/TestPatternSection/TestPatternSection.tsx`
- M `pages/DataPage/components/DataClassificationsTab/components/TestPatternSection/test-pattern-section.scss`

### Cross-PR PENDING entry (2)

- M `DataModel/DataClassificationTypes/ClassificationTypes.ts` — re-adds `PENDING: "PENDING"` to `CLASSIFICATION_STATUS` enum-as-const + adds the `PENDING` entry to `CLASSIFICATION_STATUS_CONFIG` (icon `pending`, color `uwGray02`, label `"Under evaluation"`). PR1 foundation removed both as a backward-compat measure (see foundation table below); PR4 brings them back.
- M `DataModel/DataPageModel/ClassificationsFilterConfig.ts` — single-line addition: `[CLASSIFICATION_STATUS.PENDING]: "Under evaluation"` in `CLASSIFICATION_STATUS_TRANSLATION`. Forced by PR4's `PENDING` enum addition; PR2 doesn't add it.

---

## PR 5 · Global Exclusion (9)

> Global exclusion side pane. Contains breaking type changes in `globalExclusionTypes.ts`; all producers and consumers ship together in this PR.

- M `DataModel/DataPageModel/GlobalExclusionsFilterConfig.ts`
- M `ServerAPI/Requests/DataClassification/GlobalExclusion.ts`
- M `ServerAPI/Requests/DataClassification/globalExclusionTypes.ts`
- M `sidepanes: GlobalExclusionsSidePane/GlobalExclusionsSidePane.tsx`
- M `sidepanes: GlobalExclusionsSidePane/GlobalExclusionModal/GlobalExclusionModal.tsx`
- M `sidepanes: GlobalExclusionsSidePane/GlobalExclusionsTable/GlobalExclusionActionsCell.tsx`
- M `sidepanes: GlobalExclusionsSidePane/GlobalExclusionsTable/GlobalExclusionsTable.tsx`
- M `sidepanes: GlobalExclusionsSidePane/GlobalExclusionsTable/GlobalExclusionsTableColumnDefs.ts`
- D `sidepanes: GlobalExclusionsSidePane/GlobalExclusionsTable/GlobalExclusionCapabilitiesCell.tsx`

---

## File overlap check

| PR                   | Paths           | Cross-PR overlap                                                   |
| -------------------- | --------------- | ------------------------------------------------------------------ |
| PR1 Foundation       | 24 + 1 deletion | —                                                                  |
| PR2 Main Table       | 9               | `ClassificationsFilterConfig.ts` — PR4 adds the `PENDING` line     |
| PR3 Create Sidepane  | 8               | none (TestPatternSection moved to PR4 per migration note)          |
| PR4 Detail Sidepane  | 26              | shares `ClassificationsFilterConfig.ts` (PR2) + `ClassificationTypes.ts` (PR1) — both for `PENDING` re-add |
| PR5 Global Exclusion | 9               | 0                                                                  |
