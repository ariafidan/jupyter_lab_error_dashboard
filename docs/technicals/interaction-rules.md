# UI Interaction Rules

**Owner:** Megane  
**Reviewer:** Tristan  

---

## 1. Core Controls & Expected Behavior

| Control / Element | User Input | Expected Dashboard Result |
| :--- | :--- | :--- |
| **Timeline Marker** | Hovers mouse over any 🔴/🟡/🔵/🟢 node on `TimelineTrack`. | Displays `ErrorTooltip` near cursor with timestamp, cell ID, error type, and plain-text summary from `error_catalog.json`. |
| **Timeline Marker** | Clicks an error node on `TimelineTrack`. | Selects that event (`selectedEventId`), highlights its resolution arc (if resolved), scrolls `EventListPanel` to that item, and updates the detail panels. |
| **Event List Badge** | Hovers mouse over an error badge in `EventListItem`. | Displays the same concise `ErrorTooltip` explanation. |
| **Event List Item** | Clicks a row in `EventListPanel`. | Highlights the row, highlights the matching node on `TimelineTrack`, and updates `ErrorDetailView`, `CategorizedPastSolutionsView`, and `DebuggingAssistantView`. |
| **Traffic Light Filter** | Toggles 🔴 Critical, 🟡 Warning, or 🔵 Hint chips. | Filters `EventListPanel` and dims non-matching error nodes on `TimelineTrack` (keeping timeline scale intact). |
| **Error Type Filter** | Selects an exception type (e.g., `KeyError`) or `"All"`. | Shows only matching `errorType` events. |
| **Past Solutions Tab / Accordion** | Opens `CategorizedPastSolutionsView` for the selected error. | Lists all earlier `resolved` events in the session/history with the **same `errorType`**, showing side-by-side `brokenCode` and `fixedCode` diffs. |
| **Exception Name Link** | Clicks the underlined exception title in `ErrorDetailView`. | Opens `VocabularyBoxModal` with definition, checklist, and example code from `error_catalog.json`. |
| **Preview Sandbox Fix** | Clicks "Show Fix Preview" in `SafeSandboxFixPreview`. | Reveals the diff between current broken code and the linked fix / suggested pattern. Collapsed by default. |

---

## 2. Edge-Case & Fallback Rules

### Case A: No Previous Solutions Exist for This Error Type
* **Scenario:** The learner hits a `ZeroDivisionError` for the very first time, so there are 0 prior resolved `ZeroDivisionError` records.
* **Rule:** `CategorizedPastSolutionsView` displays: *"No previous solved examples for `ZeroDivisionError` yet. Once you fix this one, it will appear here for future reference!"*

### Case B: Filtering Hides the Currently Selected Event
* **Scenario:** Active filters exclude `selectedEventId`.
* **Rule:** Automatically select the first visible error event in the filtered list. If 0 events match, set `selectedEventId = null` and show the empty-filter state with a **"Reset Filters"** button.

### Case C: Missing Code or Runtime State (Partial Juxl Records)
* **Missing `codeContext`:** Hide code pointer arrow and show *"Code snapshot not available for this record."*
* **Missing `variableSnapshot`:** Show *"Runtime variable values were not recorded for this event."*
* **Missing `fixLink`:** Mark event as `"Unresolved"` and display *"Not resolved yet"* in the fix preview panel.

### Case D: Tooltip Positioning Near Panel Edges
* **Rule:** `ErrorTooltip` automatically flips left/right or up/down if rendered within `16px` of the JupyterLab sidebar edge so text is never clipped.
