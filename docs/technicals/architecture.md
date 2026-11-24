# System Architecture & Technical Approach

**Owner:** Megane  
**Status:** Updated Draft for Team Review  

---

## 1. System Structure (Decoupled Hybrid Architecture)

The dashboard is built as a **local-first JupyterLab 4 frontend extension** (TypeScript + React) that **does not fully depend on Juxl**. While it ingests xAPI logs from `@juxl/logging` (via browser storage or exported JSON files prepared by Dicle), it also captures native JupyterLab cell execution events, full code snapshots, and static AST/dictionary explanations independently. This ensures full functionality even if Juxl logging is paused for privacy or lacks code-level diffs.

```text
+-----------------------------------------------------------------------+

|                        JupyterLab 4.6.4 Workspace                     |
|                                                                       |
|  +-----------------------+     Native Signals      +---------------+  |
|  |   Python Notebook     |------------------------>|  Direct Cell  |  |
|  |  (Learner runs code)  |                         |  & AST Hook   |  |
|  +-----------------------+                         +---------------+  |
|             |                                              |          |
|             | (Optional / Supplemental)                    |          |
|             v                                              |          |
|  +-----------------------+     Browser Storage /           |          |
|  |    @juxl/logging      |----> JSON Export (Dicle)        |          |
|  |   (xAPI Statements)   |             |                   |          |
|  +-----------------------+             v                   v          |
|                              +-------------------------------------+  |
|                              |   Data Normalizer & Local Store     |  |
|                              | (IndexedDB + error_catalog.json)    |  |
|                              +-------------------------------------+  |
|                                                |                      |
|                                                v                      |
|  +-----------------------------------------------------------------+  |
|  |                 Error Dashboard UI (Megane)                     |  |
|  |                                                                 |  |
|  |  [Interactive Timeline + Tooltips]   [Traffic Light Filters]    |  |
|  |  [Chronological Event List]          [Error Detail & Vocab Box] |  |
|  |  [Categorized Past Solutions]        [Assistant & Variables]    |  |
|  +-----------------------------------------------------------------+  |
+-----------------------------------------------------------------------+
```

### Server & Database Decision
* **Database:** **No external database required.** Browser `IndexedDB` / `localStorage` stores normalized session records and resolved error-to-fix pairs locally, paired with JSON import/export for reproducible demos.
* **Server:** **Not required for core rendering.** All filtering, error-to-fix pairing, tooltip lookups (`error_catalog.json`), and timeline calculations run client-side in TypeScript. *(Optional enhancement: a lightweight Python Jupyter Server handler using Python's built-in `ast` module for deeper syntax pointer offsets).*

---

## 2. Technical Approach & Tool Comparison

| Option | Description | Pros | Cons | Decision |
| :--- | :--- | :--- | :--- | :--- |
| **1. Decoupled JupyterLab React Extension (Recommended)** | Native JupyterLab widget reading both Juxl xAPI exports and direct `NotebookActions` signals + static `error_catalog.json`. | Zero external API cost; works offline; resilient if Juxl is paused; accesses full cell source code for diffs & past solutions. | Requires `jlpm` frontend build step. | **Selected** |
| **2. Pure Juxl-Dependent Wrapper** | Only parses `@juxl/logging` xAPI records. | Minimal capture code. | Fails when Juxl is paused; lacks full before/after cell code diffs and runtime variable values. | Rejected |
| **3. Cloud LLM-Powered Dashboard** | Sends every error to an external LLM API for explanations. | Flexible prose responses. | High latency, API costs, privacy risks, and potential spoilers. | Rejected |

### Confirmed Software & Assets
* **Environment:** JupyterLab `4.6.4` in `.venv`, `@juxl/* v4.0.0` (optional input source).
* **Frontend Stack:** TypeScript, React, JupyterLab UI tokens (`@jupyterlab/notebook`, `@jupyterlab/apputils`), and custom SVG/CSS for the interactive timeline and tooltips.
* **Static Knowledge Base:** `error_catalog.json` bundled directly with the extension for instant hover tooltips, plain-text translations, and vocabulary definitions.

---

## 3. Unresolved Decisions
1. **Direct Capture Priority vs. Juxl Import:** Confirm with Dicle whether live sessions should merge direct JupyterLab execution events with Juxl xAPI records into a single `IndexedDB` store, while keeping standalone JSON file loading for offline evaluation.
2. **Cross-Session Persistence:** Confirm with Emily whether "Previous Solutions" (Error Categorization) should persist across multiple lab sessions in `IndexedDB` or reset per study session.
