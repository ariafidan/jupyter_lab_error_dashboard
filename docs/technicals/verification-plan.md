# Verification Plan & Demonstration Script

**Owner:** Megane  
**Research Alignment:** Emily  
**Integration Partner:** Dicle  

---

## 1. Functional Verification Checklist

### Decoupled Data Loading & Ordering
- [ ] **Independent Live Capture:** With Juxl logging paused/disabled, running a failing cell in JupyterLab still records the error, full code snippet, and outcome in the dashboard.
- [ ] **Juxl JSON Import:** Loading Dicle's exported xAPI-converted JSON populates the dashboard cleanly.
- [ ] **Invalid File Handling:** Uploading malformed JSON displays a clear warning banner without losing existing session data.
- [ ] **Chronological Ordering:** All `error` and `success` events appear in ascending timestamp order.

### Timeline Visualization & Hover Tooltips
- [ ] **Outcome Markers:** Timeline displays distinct visual markers for 🔴 Critical, 🟡 Warning, 🔵 Hint, and 🟢 Success runs, plus resolution spans connecting errors to fixes.
- [ ] **Instant Hover Tooltips:** Hovering over any timeline marker or event list badge immediately shows a readable plain-language explanation from `error_catalog.json` without clipping at screen edges.
- [ ] **Bidirectional Selection:** Clicking a timeline node selects and scrolls to the corresponding item in `EventListPanel` (and vice versa).

### Error Categorization & Past Solutions
- [ ] **Same-Type Grouping:** When selecting a second `KeyError`, `CategorizedPastSolutionsView` automatically surfaces the first resolved `KeyError` with its before/after code diff.
- [ ] **First-Time Error Fallback:** Selecting an error type with no prior resolved examples shows an encouraging placeholder message instead of an empty box.

### Filtering, Detail View & Missing Data
- [ ] **Filter Auto-Reselection:** Filtering out the currently selected event automatically selects the first remaining match (or displays the "Reset Filters" empty state).
- [ ] **Vocabulary Box & Safe Fix:** Clicking the exception name opens `VocabularyBoxModal`; fix diffs remain collapsed until **"Show Fix Preview"** is clicked.
- [ ] **Partial Juxl Record Handling:** Records missing `variableSnapshot` or `codeContext` display clear fallback text without errors.

---

## 2. Research Evaluation Alignment (with Emily)

1. **Self-Reference Learning:** Verify whether learners use `CategorizedPastSolutionsView` ("How I solved this before") to resolve repeat error types faster over time.
2. **Non-Intrusive Scaffolding:** Hover tooltips and progressive-disclosure fix previews provide tiered help (Tooltip → Plain Text → Checklist → Past Solution → Fix Diff) without spoiling answers upfront.
3. **Clean Timing Metrics:** Idle gaps (>2 min) are visibly separated on the timeline and excluded from `activeTimeToFixSeconds`.

---

## 3. Demonstration Walkthrough (End-to-End Script)

### Scenario 1: Repeat Error Categorization, Timeline & Tooltip Flow
1. **First `KeyError` & Fix:** Run `student = {'name': 'Sara'}` and `print(student['alter'])`. Show the 🔴/🟡 node appear on `TimelineVisualizationPanel`, hover over it to show the instant **`ErrorTooltip`**, then fix it with `student.get('alter')` (creating a 🟢 success node and resolution span).
2. **Second `KeyError` (Past Solution Lookup):** In another cell, trigger a new `KeyError` (`grades['math']`). Select it in the dashboard and open **`CategorizedPastSolutionsView`** to demonstrate how the dashboard automatically displays the student's own solved `student['alter']` → `student.get('alter')` example.
3. **Vocabulary & Analytics:** Click `KeyError` to open `VocabularyBoxModal`, then show updated `TimeToFixSummary` metrics.

### Scenario 2: Decoupled Fallback & Missing-Data Case
1. **Pause Juxl & Trigger `SyntaxError`:** Pause Juxl logging to prove independence, then run `if True` (missing `:`).
2. **Verify Graceful Fallbacks:** Show that the dashboard still captures the 🔴 `SyntaxError` via direct JupyterLab signals, displays *"Runtime variable values were not recorded"* in `VariableStateSnapshot`, and shows the first-time placeholder in `CategorizedPastSolutionsView`.
