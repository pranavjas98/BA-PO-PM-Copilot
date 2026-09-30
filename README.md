# BA, PO & PM AI Copilot

An AI workspace for **Business Analysts, Product Owners and Project Managers**. Describe what you need in plain language and the copilot produces requirement documents, agile artifacts, BPMN diagrams and project-management deliverables. It can then stress-test them against regulations and stakeholder viewpoints, keep an immutable version history, and sync work items to JIRA.

Built with [Dash](https://dash.plotly.com/), SQLite and a multi-provider LLM failover chain (Gemini → Groq → OpenRouter → local Ollama).



Demo

Watch the walkthrough of the copilot in action: generating a requirements document, running Regulatory Radar and a stakeholder Red-Team review, applying remediation, and syncing to JIRA.



https://github.com/user-attachments/assets/311df82d-4236-45c5-82dc-8abf00612239




---

## Table of contents

- [Features](#features)
- [Modes by role](#modes-by-role)
- [How it works](#how-it-works)
- [Tech stack](#tech-stack)
- [Getting started](#getting-started)
- [Configuration](#configuration)
- [Usage guide](#usage-guide)
- [Project structure](#project-structure)
- [Data and privacy](#data-and-privacy)
- [Troubleshooting](#troubleshooting)
- [Limitations](#limitations)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)

---

## Features

### Document and artifact generation
- **Requirement documents:** BRD, FRD, PRD, SRS, FSD, SOP and SOW, each with its own structure and terminology.
- **Document conversion:** convert one type to another (for example BRD → FRD). The conversion rewrites the structure rather than just renaming headings, preserves requirement IDs, and never overwrites the original.
- **User stories and backlog:** epics, stories, acceptance criteria (Given/When/Then), story points and priorities rendered as cards.
- **BPMN 2.0 diagrams:** generated from a process description, rendered in-browser with bpmn.js, exportable as `.bpmn`, SVG or PNG.
- **Analytics, RTM and stakeholder emails:** insights with auto-charts, requirements traceability matrices and meeting summaries.

### Quality and compliance
- **Regulatory Radar:** detects the likely domain and jurisdiction, lists applicable regulations and gaps, and lets you apply all or selected findings.
- **Stakeholder Red-Team:** critiques the artifact from multiple personas (Compliance Officer, Developer, QA, Security, Architect, Sponsor and more). A recommended persona set is preselected for each document type, and you can add a custom stakeholder.
- **Remediation preview:** every fix shows a before/after view and a unified diff first. Nothing changes until you click **Apply Changes**.
- **Compliance Dashboard:** verified before/after findings, severity distribution, remediation score and version history timeline.

### Workflow and governance
- **Immutable workspace versions:** every generation, conversion and remediation creates a new version with parent lineage. Reopen any version from the chat.
- **Scope Management (PM):** scope baseline, WBS, requirements-to-scope traceability, scope change analysis and a scope-creep detector.
- **Change Ripple:** link related artifacts. When one changes, linked artifacts are flagged for review, and similar chats are suggested.
- **Meeting → Artifacts pipeline:** paste a transcript and get a BRD, user stories and (if a process is described) a BPMN model, automatically linked.
- **JIRA sync:** push epics, stories and tasks with sub-tasks, epic linking, update-in-place and a "changed since sync" indicator.

### Platform
- **Resilient AI failover:** if a model is rate-limited or down, the app moves to the next one, with per-model cooldowns.
- **Persistent conversations:** chats are grouped into projects, can be archived, renamed or moved, and long chats are summarised automatically to keep context.
- **File attachments:** `.txt`, `.md`, `.csv`, `.xlsx`, `.pdf`, `.docx`, code and JSON files can be attached to a prompt.
- **Export:** Word, PDF, plain text, Markdown, BPMN XML, SVG and PNG.

---

## Modes by role

| Role | Modes |
|------|-------|
| **Business Analysis** | General, Requirements Docs (BRD/FRD/PRD/SRS/FSD/SOP/SOW), Stories, Analytics, Process (text or BPMN), RTM, Email |
| **Product Ownership** | General, Backlog, Prioritize (WSJF/MoSCoW), PI Planning, Sprint, Roadmap, Vision (OKRs), Acceptance criteria |
| **Project Management** | General, Scope Management, Charter, PM Plan, RAID Log, Status Report, Governance, Change Log, Closure |

The copilot is industry-agnostic by default. It only adapts to a domain (banking, insurance, healthcare, SaaS and so on) when your request or attachments make the domain clear.

---

## How it works

```
User prompt ──► Role + mode prompt ──► AIProvider ──► Artifact
                                            │             │
                          Gemini ► Groq ► OpenRouter ► Ollama
                                                          │
                       ┌──────────────────────────────────┤
                       ▼                  ▼               ▼
                Workspace view     Version history    JIRA / Export
                       │
        Regulatory Radar · Red-Team · Conversion
                       │
             Preview diff ──► Apply ──► new immutable version
```

- **Structured output** (cards, BPMN, scope) uses native JSON modes where available, with strict-prompt fallbacks and a JSON repair step.
- **Free-form documents** are remediated section by section so large documents are not truncated.
- **Verification:** after a remediation is applied, the new version is re-scanned and the burden is compared with the baseline. The severity-weighted burden is a comparison metric, not a compliance certification.

---

## Tech stack

| Layer | Technology |
|-------|-----------|
| UI | Dash, Dash Bootstrap Components, Plotly, bpmn.js |
| Storage | SQLite |
| LLM providers | Google Gemini, Groq, OpenRouter, Ollama (local) |
| Integrations | JIRA Cloud REST API v3 |
| Export | python-docx, ReportLab |
| File parsing | pandas, openpyxl, pypdf |

---

## Getting started

### Prerequisites
- Python 3.10 or newer
- At least one AI provider key (Gemini, Groq or OpenRouter), or a local [Ollama](https://ollama.com/) install
- Jupyter (Anaconda, VS Code or JupyterLab) to run the notebook

### Installation

```bash
git clone https://github.com/pranavjas98/BA-PO-PM-Copilot.git
cd BA-PO-PM-Copilot
pip install -r requirements.txt
```

### Set your API keys

The app reads keys from environment variables. They are never stored in the source code.

**Windows (PowerShell)**

```powershell
setx GEMINI_API_KEY "your-key"
setx GROQ_API_KEY "your-key"
setx OPENROUTER_API_KEY "your-key"
```

**macOS / Linux**

```bash
export GEMINI_API_KEY="your-key"
export GROQ_API_KEY="your-key"
export OPENROUTER_API_KEY="your-key"
```

After using `setx`, close and reopen your terminal and restart Jupyter so the new values are picked up.

### Run

1. Open `ba_pm_po_ai_copilot.ipynb` in Jupyter.
2. Run all cells.
3. The app opens at **http://127.0.0.1:8051**. If it does not, open that address manually.

On startup the console prints which API keys were detected.

---

## Configuration

| Variable | Required | Purpose |
|----------|----------|---------|
| `GEMINI_API_KEY` | One provider needed | Primary provider |
| `GROQ_API_KEY` | Optional | First fallback |
| `OPENROUTER_API_KEY` | Optional | Second fallback (free models are auto-discovered) |
| `BA_PO_DB_PATH` | Optional | SQLite file location. Defaults to `ba_po_workspace.db` in the working directory |
| `JIRA_BASE_URL` | For JIRA | For example `https://your-team.atlassian.net` |
| `JIRA_EMAIL` | For JIRA | Atlassian account email |
| `JIRA_API_TOKEN` | For JIRA | [Atlassian API token](https://id.atlassian.com/manage-profile/security/api-tokens) |

See `.env.example` for a template. JIRA features stay disabled until all three JIRA variables are set.

### Choosing models

Model pools are defined at the top of the notebook and tried in order:

- `GEMINI_PREFERRED_MODELS`
- `GROQ_PREFERRED_MODELS` and `GROQ_JSON_MODELS`
- `OPENROUTER_PREFERRED_MODELS`
- `OLLAMA_QWEN_MODEL` for the local last-resort fallback

Edit these lists to match the models available to your accounts. Model names change often, so update them if a provider reports a model as unavailable.

### Optional: local fallback with Ollama

```bash
ollama pull <your-model>
```

Set `OLLAMA_QWEN_MODEL` to that model name. Ollama is called at `http://localhost:11434` only if every cloud provider fails.

---

## Usage guide

### Generate a document
1. Choose a role (BA, PO or PM) and a mode.
2. Type your request, for example: *"Create an FRD for a patient appointment scheduling system."*
3. The result opens in the **Workspace** panel. Each generation is saved as a version.

For requirement documents, name the type in your prompt (BRD, FRD, PRD, SRS, FSD, SOP or SOW). If you do not, BRD is used.

### Convert a document
Open a requirements document, click **🔄 Convert**, choose a target type and confirm. A new version is created and the original is left unchanged.

### Run a Regulatory Radar scan
1. Open an artifact and click **🛡️ Regulatory Radar**, then **Run Scan**.
2. Review the findings and tick the ones to fix.
3. Click **Apply All Findings** or **Apply Selected**, review the diff, then **Apply Changes**.

### Run a stakeholder Red-Team review
1. Click **🎭 Red-Team**. The recommended stakeholders for the document type are preselected.
2. Adjust the selection, or add a custom stakeholder.
3. Click **Run Critique**, then apply all or selected concerns after previewing the diff.

### Check compliance progress
Click **📊 Compliance Dashboard** to see the current status, before/after finding counts, severity distribution and the verified version timeline.

### Push work items to JIRA
Stories, backlog, prioritization, RAID, change log and scope outputs offer a **🚀 Push to JIRA** flow. Pick a space, and the app creates or updates issues, sub-tasks and epic links.

### Manage scope
Use PM → **Scope Management** to generate a baseline. Then use **🔀 Scope Change** to analyse a requested change and **🔎 Scope Creep** to compare JIRA issues labelled `copilot-scope` against the baseline and WBS.

### Turn a meeting into artifacts
Click **🎙️ Meeting → Artifacts**, paste a transcript or attach a file, and generate. Results are saved as separate chats you can open from **📜 History**.

---

## Project structure

```
BA-PO-PM-Copilot/
├── ba_pm_po_ai_copilot.ipynb   # Full application (Dash app, DB layer, AI provider, callbacks)
├── requirements.txt            # Python dependencies
├── .env.example                # Environment variable template
├── .gitignore                  # Keeps databases, exports and secrets out of Git
├── LICENSE                     # MIT
└── README.md
```

Inside the notebook, the main sections are:

| Section | Responsibility |
|---------|----------------|
| Configuration | Model pools, environment variables, provider clients |
| SQLite layer | Chats, projects, messages, workspace artifacts, JIRA sync, ripple links |
| `AIProvider` | Provider failover, JSON mode and message construction |
| Artifact rendering | Documents, cards, BPMN and scope views |
| Regulatory Radar and Red-Team | Scans, remediation, preview and verification |
| Callbacks | UI behaviour, export, history and modals |

---

## Data and privacy

- All chats, artifacts and scan results are stored **locally** in a SQLite file. Nothing is uploaded except the content sent to the AI providers you configure.
- Prompts and attached documents are sent to those providers, so do not use confidential material unless your provider agreements allow it.
- The `.gitignore` excludes `*.db`, `*.xlsx` and `*.csv` so your working data is not published by accident.
- Never commit API keys. If one is exposed, rotate it immediately.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `API key missing` in the console | Set the environment variable, then restart the terminal and Jupyter |
| "All AI providers exhausted" | Check keys and quotas, and update the model lists to currently available models |
| JIRA push does nothing | Confirm `JIRA_BASE_URL`, `JIRA_EMAIL` and `JIRA_API_TOKEN` are all set |
| Port 8051 already in use | Stop the earlier run (restart the kernel) or change the `port` in `app.run(...)` |
| BPMN diagram is blank | Confirm the browser can load the bpmn.js script from `unpkg.com` |
| Remediation looks truncated | The app rejects truncated documents. Retry, or apply fewer findings at a time |
| `git` command not found | Install Git and reopen your terminal |

---

## Limitations

- **Not legal advice.** Regulatory Radar is an AI-assisted, informational scan. It can miss or misjudge regulations. Have compliance or legal counsel review anything that matters.
- AI output can be wrong or incomplete. Review every artifact before relying on it.
- The remediation score compares severity-weighted findings between versions. It is not a certification.
- Free-tier models can be slow, rate-limited or inconsistent in structured output.
- The app is designed for single-user local use and has no authentication.

---

## Roadmap

- [ ] Standalone `app.py` entry point alongside the notebook
- [ ] Automated tests for parsing, versioning and remediation
- [ ] Docker image
- [ ] Multi-user support with authentication
- [ ] Additional integrations (Confluence, Azure DevOps)
- [ ] Template library for organisation-specific document formats

---

## Contributing

Issues and pull requests are welcome.

1. Fork the repository and create a feature branch.
2. Clear all notebook outputs before committing (`Kernel → Restart & Clear Output`).
3. Do not include API keys, databases or personal paths.
4. Open a pull request describing the change and how you tested it.

---

## License

Released under the [MIT License](LICENSE).

---

## Author

**Pranav Jaswal**
[GitHub](https://github.com/pranavjas98)
