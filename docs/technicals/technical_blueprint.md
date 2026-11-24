# Technical & Architectural Blueprint: Pedagogical Python Error Dashboard

## 1. System Architecture Overview

This JupyterLab extension follows a **local-first, deterministic architecture** designed to provide instant pedagogical feedback without relying heavily on cloud LLMs. It combines JupyterLab's native notebook signals, **Juxl** for standardized activity logging, silent **Python Kernel inspection**, and Python's built-in **AST/tokenization tools**.

```text
[Student executes a code cell]
       │
       ├──► Juxl (@juxl/*) logs standardized xAPI event (Background Analytics)
       │
       └──► Dashboard Frontend intercepts NotebookActions.executed signal
                  │
                  ├──► 1. Silent Kernel Query ──► Captures live variable types & values
                  │
                  └──► 2. Local Python Server Endpoint (/analyze)
                            ├── ast.parse() & tokenize  ──► Syntax & Best-Practice Rules
                            ├── difflib & Scope Check   ──► Typo & Execution-Order Hints
                            └── Error Template Engine   ──► Plain-Text + Checklist + Diff
```

---

## 2. Installed Components & Their Roles

The following tools and packages are already installed in the local virtual environment (`.venv`):

| Component / Package | Version / Source | Architectural Role |
| :--- | :--- | :--- |
| **Python Virtual Environment (`.venv`)** | Python 3.11 / 3.14 | Isolates all project dependencies, JupyterLab builds, and server extensions from the global Windows system. |
| **`jupyterlab`** | `4.6.4` | Core web-based Interactive Learning Environment (ILE) hosting the notebook editor, kernels, and extension slots. |
| **`@juxl/juxl-extension`** | `4.0.0` | Core JupyterLab xAPI Logging Interface; manages privacy consent modes (*anonymous*, *pseudonymous*, *personalized*) and logging controls. |
| **`@juxl/logging`** | `4.0.0` | Event listener layer that captures notebook interactions (cell executions, edits, errors) and converts them into standardized xAPI records. |
| **`@juxl/log-console`** | `4.0.0` | Developer inspection widget for viewing live xAPI event streams directly inside JupyterLab during testing. |
| **`Node.js`, `npm`, & `jlpm`** | System / Built-in | Compiles TypeScript/React frontend code and bundles JupyterLab frontend assets (`jupyter lab build`). |
| **`deep-equal` & `@types/react` (deduped)** | Staging Build Fixes | Resolves frontend dependency and TypeScript type collisions during JupyterLab extension compilation. |

---

## 3. Packages to Install Next (By Development Phase)

### Phase A: Extension Scaffolding & Backend Validation (Install Now)
Run inside the active `.venv` before generating the extension skeleton:

* **`copier` & `jinja2-time`** (`pip install copier jinja2-time`)
  * **Role:** Official JupyterLab 4 extension template generator used to scaffold the frontend TypeScript plugin and Python server extension structure.
* **`pydantic`** (`pip install pydantic`)
  * **Role:** Enforces strict JSON data schemas between the Python analysis backend and the React dashboard UI.

### Phase B: Python Standard Library Modules (Zero Installation Required)
Built directly into Python and used as the core analysis engine:

* **`ast` (Abstract Syntax Tree):** Parses student code into a syntax tree to detect structural errors, variable scopes, definition line numbers, and best-practice anti-patterns.
* **`tokenize`:** Breaks source lines into exact lexical tokens (`NAME`, `OP`, `NEWLINE`, `INDENT`) to pinpoint missing colons, unclosed brackets, and exact column offsets.
* **`difflib`:** Performs fuzzy string matching (`get_close_matches`) to catch typos in variable names, function calls, and dictionary keys.

### Phase C: Optional Future Additions (Install Later)
* **`lucide-react`** (`jlpm add lucide-react`): Lightweight icon set for traffic light indicators (🔴/🟡/🔵), visual pointers, and milestone badges.
* **`recharts`** (`jlpm add recharts`): React charting library for the long-term "Most Frequent Errors" donut chart and "Time-to-Fix" tracker (deferred until core debugging features are complete).
* **`openai` or `google-genai` (Optional LLM Fallback):** Only needed if an optional *"Ask AI for deeper explanation"* button is added later for rare, ambiguous runtime bugs that deterministic rules cannot cover.

---

## 4. How AST & Deterministic Methods Replace the LLM

By combining Python's built-in `ast`, `tokenize`, `difflib`, and a silent **Kernel Variable Snapshot**, the dashboard delivers almost every pedagogical feature in `Design.md` with **0 ms latency**, **zero API cost**, and **no hallucinations**:

### A. Plain-Text Translator & Visual Token Pointer (`ast` + `tokenize`)
* **Syntax & Indentation Errors (🔴 Critical):**
  * When `ast.parse(code)` raises a `SyntaxError` or `IndentationError`, Python provides the exact `lineno`, `offset` (column), and `end_offset`.
  * Using `tokenize`, the backend inspects the tokens immediately preceding `offset` (for example, detecting an `if`, `for`, `while`, or `def` statement that lacks a trailing `:`) and triggers a deterministic rule: *"You started an `if` condition on line 3, so it needs a colon `:` at the end."*
  * The exact line and column coordinates drive the UI's **Visual Pointer arrow** directly beneath the faulty or missing token.
* **Runtime Errors (🟡 Warning):**
  * Maps specific exception classes (`KeyError`, `TypeError`, `IndexError`, `ZeroDivisionError`, `AttributeError`) to structured plain-text explanation templates dynamically populated with the exception value (`evalue`) and live variable values.

### B. Guided Checklists & Typo Detection (`ast` + `difflib`)
* **`NameError` ("Did you make a typo?"):**
  * Walks the cell's AST (`ast.walk(tree)`) to collect all defined variable and function symbols (`ast.Name` in `Store` context, `ast.FunctionDef`), merges them with active variable names from the kernel snapshot and Python built-ins, and runs `difflib.get_close_matches()`.
  * *Typo match:* If a learner writes `pritn("hi")`, `difflib` immediately suggests: *"Did you mean `print`?"*
  * *Order-of-definition check:* If the variable exists in the AST on a *later* line than where it was called, the analyzer reports: *"You defined `total` on line 8, which is after you tried to use it on line 2."*
* **`KeyError` ("Closest matching key"):**
  * Compares the missing key against `list(target_dict.keys())` retrieved from the kernel snapshot using `difflib` to suggest: *"Key `'alter'` was not found in this dictionary. Did you mean `'age'`?"*

### C. Safe Sandbox Fixes (Deterministic Token & AST Patching)
* For common beginner syntax patterns—such as a missing colon `:`, an unclosed parenthesis `(`, or accidentally using assignment `=` instead of comparison `==` inside an `if` condition—the token/AST analyzer constructs the exact single-line corrected code preview deterministically without calling an LLM.

### D. Best-Practice Logical Hints (🔵 AST Linter Rules)
* Even when a cell executes without throwing an error, a custom `ast.NodeVisitor` scans the syntax tree to surface clean-code hints:
  * **Overly broad exception handling:** Detects bare `except:` or `except Exception:` (`ast.ExceptHandler` where `node.type is None` or `node.type.id == "Exception"`).
  * **Mutable default arguments:** Flags function definitions such as `def add_item(items=[]):` where `ast.FunctionDef.args.defaults` contains a `List`, `Dict`, or `Set` literal.
  * **Shadowing Python built-ins:** Flags assignments that overwrite built-in names (e.g., `list = [1, 2, 3]` or `str = "hello"`).

### E. Runtime State Awareness (Silent Kernel Variable Inspector)
* Because static AST analysis cannot know runtime values (e.g., `x = random.randint(1, 100)`), the frontend executes a silent background query (`kernel.requestExecute` with `silent: true`) upon any runtime crash.
* This extracts a safe, truncated JSON snapshot of user-defined variables (`name`, `type`, and `repr(value)`) from `globals()`, feeding both the **Variable Inspector UI** and the deterministic error templates.
