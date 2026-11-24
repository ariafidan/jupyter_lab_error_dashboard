# Pedagogical Error Dashboard Design (Python ILE)

An error dashboard in an Integrated Learning Environment (ILE) for Python differs fundamentally from professional monitoring tools like Sentry or AppSignal. While those are optimized for developers focusing on system stability, a dashboard for learners needs to be pedagogical, motivating, and easy to understand. Its goal is to reduce frustration and build coding competence (debugging skills).

An effective ILE error dashboard for Python learners can be divided into three core functional areas:

## 1. The Error Detail View (The "Translator")

Standard Python tracebacks often overwhelm beginners because they appear cryptic. The detail view visually prepares the current error:

* **The "Plain Text" Translator:** Instead of just displaying `KeyError: 'alter'`, the dashboard translates it into: *"You are trying to access the key 'alter' in a dictionary, but this key does not exist there."*
* **Visual Pointer (Code Context):** The dashboard highlights the affected line in the code editor. A small arrow points directly to the faulty token (e.g., a missing closing parenthesis in a `SyntaxError`).
* **Interactive Vocabulary Box:** Every displayed exception (e.g., `TypeError`, `ValueError`) is clickable and opens a short, visual mini-glossary explaining what that specific error type means at its core.

## 2. The Debugging Assistant (Help for Self-Help)

The dashboard must not simply give away the solution, but rather guide the learning process:

* **Guided Checklists:** Dynamic questions that encourage critical thinking. For a `NameError`, it might ask: *"Did you make a typo? Did you define the variable earlier in your code? Was the line containing the definition actually executed?"*
* **Safe Sandbox Fixes:** Suggestions for typical error patterns (e.g., a missing colon `:` in `if` statements). With one click, the learner can preview what the corrected code snippet would look like before applying it themselves.
* **Variable Inspector (State Snapshot):** A simplified view showing the exact values the variables held at the moment of the crash. This makes it immediately visible if a variable unintentionally contained `None` or the wrong data type.

---

## 3. Error Categorization & Past Solutions ("How I Solved This Before")

Learners frequently repeat the same category of mistakes across a session or semester. Rather than treating every crash in isolation:

* **Grouped Error Categories:** Errors are automatically categorized by Traffic Light severity and Python exception type (e.g., `KeyError`, `IndentationError`, `TypeError`).
* **Previous Solutions Browser:** When a learner encounters an error type they have successfully fixed earlier, the dashboard surfaces those **past error-to-fix pairs** (showing their previous broken code alongside the code that fixed it), turning their own history into a personalized reference guide.

---

## 4. The Progress & Analytics Overview (Long-Term View)

This component shows the learner (and optionally instructors) recurring patterns over an extended period:

| Dashboard Element | Purpose for Learners | Benefit for the Learning Process |
| :--- | :--- | :--- |
| **The "Most Frequent Errors" Pipeline** | Chart (e.g., a donut chart) of the top 3 error types of the week (e.g., 40% `IndentationError`, 30% `AttributeError`). | Sharpens awareness of personal "careless mistakes" while writing code. |
| **The "Time-to-Fix" Tracker** | Displays how long the user takes on average to resolve a specific type of error. | Visualizes learning progress: when the time spent fixing a `KeyError` drops, the user understands the concept better. |
| **Milestones & Badges** | Rewards such as *"Syntax-Safe: 10 code submissions without an indentation error!"* | Transforms errors from moments of frustration into gamification points ("learning from mistakes"). |

## Pedagogical Design Principle: The Traffic Light System

To avoid visual overwhelm, modern ILE dashboards often use color-coded categorization:

1. 🔴 **Critical (Compile-Time):** `SyntaxError` or `IndentationError`. The code cannot even start. The dashboard gently blocks execution and asks for a repair first.
2. 🟡 **Warning (Runtime/Semantics):** `ZeroDivisionError` or `IndexError`. The code is syntactically correct, but crashes during execution. Here, the dashboard offers the Variable Inspector.
3. 🔵 **Logical Hint (Best Practice):** The code runs, but contains poor style (e.g., an overly broad `except Exception:` without a specific error type). Here, the dashboard suggests optimizations for clean error handling.

18.11.2026

LLM
we could use Parsing methods and AST instead of LLMs
Our dashboard must not fully depend on Juxl, we can also implement some other things we need that are not integrated in Juxl.

Error categorisation ( visualise previous error of the same type to see how it was solved), Visualisation of the timeline, tooltip with error explanations.
What type of datas 

Research protocol: Introduction on the papers, Related work, the methodology how we are going to evaluate the dashboard.
