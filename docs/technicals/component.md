# UI Component Specification

**Owner:** Megane  
**Design Reference:** `Design.md`  

Each screen area in `Design.md` maps to a modular React component with a single responsibility.

---

## 1. Component Hierarchy

```text
ErrorDashboardWidget (Root Container)
├── DataLoaderBar
├── TimelineVisualizationPanel
│   ├── TimelineTrack (Error, Success, Idle & Resolution Span markers)
│   └── ErrorTooltip (Hover explanation overlay)
├── FilterControlBar
│   ├── TrafficLightFilter (🔴 Critical, 🟡 Warning, 🔵 Hint)
│   └── ErrorTypeFilter (Dropdown by exception type)
├── MainWorkspace (Master-Detail Split View)
│   ├── EventListPanel
│   │   └── EventListItem (with hover ErrorTooltip & Resolved badge)
│   └── DetailInspectorPanel
│       ├── ErrorDetailView
│       │   ├── PlainTextTranslator
│       │   ├── CodeContextPointer
│       │   └── VocabularyBoxModal
│       ├── CategorizedPastSolutionsView ("How You Solved This Before")
│       └── DebuggingAssistantView
│           ├── GuidedChecklist
│           ├── SafeSandboxFixPreview
│           └── VariableStateSnapshot
└── ProgressAnalyticsDrawer
    ├── FrequentErrorsSummary
    ├── TimeToFixSummary
    └── MilestoneBadges
```

---

## 2. Component Specifications

### A. Data Loading & Timeline Visualization
* **`DataLoaderBar`**
  * **Purpose:** Loads records from Dicle's exported JSON file, Juxl browser storage, or direct JupyterLab session capture. Shows active source mode and validation status.
* **`TimelineVisualizationPanel` & `TimelineTrack`**
  * **Purpose:** Visualizes the chronological sequence of all cell runs (`critical` 🔴, `warning` 🟡, `hint` 🔵, and `success` 🟢) along a time axis. Displays **resolution spans** (connecting an error to its fixing run) and **idle gap markers** (>2 min inactivity) so learners see their debugging rhythm.
* **`ErrorTooltip`**
  * **Purpose:** Reusable hover overlay attached to timeline markers, event list badges, and category tags. Reads from `error_catalog.json` to display an instant 1–2 sentence plain-language explanation on hover.

### B. Filtering & Selection
* **`TrafficLightFilter` & `ErrorTypeFilter`**
  * **Purpose:** Filters both `TimelineTrack` and `EventListPanel` by Traffic Light severity or specific Python exception (`KeyError`, `SyntaxError`, etc.).
* **`EventListPanel` & `EventListItem`**
  * **Purpose:** Scrollable list of recorded errors showing timestamp, cell label, exception badge (with hover `ErrorTooltip`), and resolution status. Manages `selectedEventId`.

### C. Error Detail View ("The Translator")
* **`PlainTextTranslator`**
  * **Purpose:** Displays beginner-friendly explanation text from the event record or `error_catalog.json`.
* **`CodeContextPointer`**
  * **Purpose:** Displays the cell's source code at crash time, highlighting `errorLine` and placing a visual pointer (`^`) at `columnOffset`.
* **`VocabularyBoxModal`**
  * **Purpose:** Opens on clicking an exception name to display its plain-language definition, common causes, and a short before/after example.

### D. Error Categorization & Debugging Assistant
* **`CategorizedPastSolutionsView` (New)**
  * **Purpose:** Automatically filters the learner's history for **previously resolved errors of the same `errorType`** (and optional `errorSubPattern`). Displays collapsible cards showing their own past `brokenCode` vs. `fixedCode` diff so they can learn from past successes.
* **`GuidedChecklist`**
  * **Purpose:** Interactive self-reflection questions tailored to the selected `errorType`.
* **`SafeSandboxFixPreview`**
  * **Purpose:** Displays a click-to-reveal diff preview of the linked fix event or deterministic pattern fix without modifying the notebook automatically.
* **`VariableStateSnapshot`**
  * **Purpose:** Displays variable names, data types, and values captured from the kernel at the moment of a runtime crash.

### E. Progress & Analytics
* **`FrequentErrorsSummary`, `TimeToFixSummary`, & `MilestoneBadges`**
  * **Purpose:** Aggregates Top-3 error categories, average active resolution time (excluding idle gaps), and earned learning milestones.
