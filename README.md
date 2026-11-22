<!-- # jupyter_lab_extension_not_named_yet -->

# Pedagogical Python Error Dashboard for JupyterLab

A learner-focused Error Dashboard extension for **JupyterLab 4** designed to reduce beginner frustration and build debugging competence. Unlike production monitoring tools, this dashboard translates Python errors into plain language, provides interactive hover tooltips, visualizes session timelines, and surfaces a learner's own previously solved errors of the same type.

---

## Project Structure

```text
jupyter_lab_error_dashboard/
├── Design.md                         # Core pedagogical & UI design concept
├── README.md                         # Setup guide & repository overview
├── .gitignore                        # Excluded build outputs, caches & virtual environments
└── docs/
    └── technical/
        ├── architecture.md           # System structure & decoupled technical approach
        ├── components.md             # Modular React UI component specifications
        ├── data-contract.md          # Shared JSON schema (xAPI / direct capture)
        ├── interaction-rules.md      # UI state transitions & edge-case rules
        ├── implementation-plan.md    # Task breakdown & team ownership
        ├── verification-plan.md      # Functional checklist & demo script
        └── fixtures/
            ├── error_catalog.json    # Static beginner error tooltips & glossary
            └── sample_session.json   # Shared test session fixture
```

---

## Team Roles & Ownership

* **Megane:** Technical blueprint, dashboard UI components, timeline & tooltip interactions, direct JupyterLab execution capture.
* **Dicle:** xAPI log import/normalization (`@juxl/logging`), error-to-fix record pairing, joint data contract maintenance.
* **Tristan:** UI prototype design & interaction rules review.
* **Emily:** Pedagogical evaluation & study alignment.

---

## Local Setup Guide (Windows / `cmd.exe`)

### Prerequisites
Make sure the following tools are installed and available in your system `PATH`:
* **Git**
* **Python 3.11** (installable side-by-side alongside newer Python versions via the Windows Python Launcher `py`)
* **Node.js (v18 or v20 LTS)** & `npm`

### 1. Clone Repository & Create Virtual Environment
Open **Windows Command Prompt (`cmd`)** (not PowerShell) and run:

```cmd
git clone https://github.com
cd jupyter_lab_error_dashboard

py -3.11 -m venv .venv
.venv\Scripts\activate
python -m pip install --upgrade pip
```

### 2. Install JupyterLab 4.6.4 & Juxl Extensions
Install JupyterLab and the `@juxl` logging extensions (`--no-build` defers compilation until dependency conflicts are resolved):

```cmd
pip install jupyterlab==4.6.4
jupyter labextension install @juxl/juxl-extension@4.0.0 @juxl/logging@4.0.0 @juxl/log-console@4.0.0 --no-build
```

### 3. Build Frontend Assets & Resolve Dependency Conflicts
Run the initial build and apply the staging fixes for `deep-equal` and `@types/react`:

```cmd
jupyter lab build --dev-build=False --minimize=False

npm install --prefix "%VIRTUAL_ENV%\share\jupyter\lab\staging" deep-equal --no-save
copy /Y "%VIRTUAL_ENV%\Lib\site-packages\jupyterlab\staging\yarn.lock" "%VIRTUAL_ENV%\share\jupyter\lab\staging\yarn.lock"
jupyter lab build --dev-build=False --minimize=False

pushd "%VIRTUAL_ENV%\share\jupyter\lab\staging"
jlpm dedupe @types/react
popd

jupyter lab build --dev-build=False --minimize=False
```

### 4. Verify Installation
Confirm all extensions are enabled without errors:

```cmd
jupyter labextension list
```

---

## Starting JupyterLab

Whenever you return to work on the project, open `cmd` inside the repository folder, activate `.venv`, and start JupyterLab:

```cmd
.venv\Scripts\activate
jupyter lab
```

*(Tip: If browser caching hides newly built extensions after a rebuild, press `Ctrl + Shift + R` in your browser to perform a hard refresh).*

---

## Documentation & Handover Status

* **Current Phase:** Technical Planning & Architecture Handover (`docs/technical/`).
* **Next Phase:** Extension scaffolding (`copier`), parallel implementation of Dicle's xAPI normalizer and Megane's React dashboard components against `docs/technical/data-contract.md`.
