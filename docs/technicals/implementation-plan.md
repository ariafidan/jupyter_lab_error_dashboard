# Implementation Plan & Task Ownership

**Frontend / UI & Direct Capture Owner:** Megane  
**xAPI Import & Data Normalization Owner:** Dicle  
**Interaction Reviewer:** Tristan  
**Evaluation Alignment:** Emily  

---

## Phase 1: Joint Contract & Knowledge Base (Shared)

| ID | Task | Owner | Dependency | Observable Completion Condition |
| :--- | :--- | :--- | :--- | :--- |
| **T1.1** | Finalize `data-contract.md` (v1.1) and create `sample_session.json` | **Dicle & Megane** | None | `sample_session.json` includes both `error` and `success` events, at least 2 `KeyError`s (one resolved, one new) to test Past Solutions, and 1 missing-data record. |
| **T1.2** | Create static `error_catalog.json` for top 12 Python beginner errors | **Megane** | None | JSON file covers `SyntaxError`, `IndentationError`, `NameError`, `TypeError`, `ValueError`, `KeyError`, `IndexError`, `AttributeError`, `ZeroDivisionError`, `ImportError`, `FileNotFoundError`, and `UnboundLocalError`. |
| **T1.3** | Review `interaction-rules.md` (Timeline, Tooltips, Past Solutions) | **Megane & Tristan** | `components.md` | Tristan signs off on timeline interactions, hover tooltips, and past-solution cards. |

---

## Phase 2: Data Capture & Import Pipelines (Parallel)

| ID | Task | Owner | Dependency | Observable Completion Condition |
| :--- | :--- | :--- | :--- | :--- |
| **T2.1** | Build Juxl xAPI-to-Data-Contract mapper | **Dicle** | T1.1 | Converts raw `@juxl/logging` xAPI records into valid v1.1 JSON payloads with `fixLink` pairing. |
| **T2.2** | Build direct JupyterLab cell listener (`NotebookActions.executed`) | **Megane** | T1.1 | Captures `error` and `success` runs, full `sourceLines`, and error-to-fix diffs directly into `IndexedDB` even when Juxl is off. |

---

## Phase 3: Dashboard UI Components (Megane)

| ID | Task | Owner | Dependency | Observable Completion Condition |
| :--- | :--- | :--- | :--- | :--- |
| **T3.1** | Build `DataLoaderBar` & hybrid store (`IndexedDB` + JSON upload) | **Megane** | T1.1, T2.2 | Loads both live captured runs and Dicle's imported JSON files; validates schema and handles errors gracefully. |
| **T3.2** | Build `TimelineVisualizationPanel` & `ErrorTooltip` | **Megane** | T1.2, T3.1 | Renders chronological 🔴/🟡/🔵/🟢 markers, idle gaps, resolution spans, and instant hover tooltips from `error_catalog.json`. |
| **T3.3** | Build `EventListPanel`, `TrafficLightFilter`, & `ErrorTypeFilter` | **Megane** | T3.2 | Synchronizes selection with timeline; filters list and dims non-matching timeline nodes; handles empty states. |
| **T3.4** | Build `ErrorDetailView` (Translator, Pointer, `VocabularyBoxModal`) | **Megane** | T1.2, T3.3 | Displays plain translation, code highlight with column arrow, and modal glossary on exception click. |
| **T3.5** | Build `CategorizedPastSolutionsView` & `DebuggingAssistantView` | **Megane** | T3.3 | Queries resolved events of the same `errorType` to display past before/after code fixes alongside checklist and variable snapshot. |
| **T3.6** | Build `ProgressAnalyticsDrawer` (Top Errors, Time-to-Fix, Badges) | **Megane** | T3.3 | Computes Top-3 errors, active `timeToFixSeconds` (excluding idle gaps), and milestone badges. |

---

## Phase 4: Integration & Demo Verification (Shared)

| ID | Task | Owner | Dependency | Observable Completion Condition |
| :--- | :--- | :--- | :--- | :--- |
| **T4.1** | Test importing Dicle's Juxl JSON alongside live notebook runs | **Dicle & Megane** | T2.1, T3.5 | Both data sources render identically in Timeline, Tooltips, and Past Solutions. |
| **T4.2** | Execute full verification checklist & demo rehearsal | **Dicle & Megane** | T4.1 | All checks in `verification-plan.md` pass. |
