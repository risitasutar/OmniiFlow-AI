<div align="center">

# ✨ OmniFlow AI

### One AI. Every Task. Seamless Execution.

![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)
![LangGraph](https://img.shields.io/badge/LangGraph-1.2-1C3C3C?logo=langchain&logoColor=white)
![Gemini](https://img.shields.io/badge/Google%20Gemini-LLM-8E75B2?logo=googlegemini&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-1.65-FF4B4B?logo=streamlit&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-read--only-003B57?logo=sqlite&logoColor=white)
![Tests](https://img.shields.io/badge/tests-726%20offline-2EA44F?logo=pytest&logoColor=white)
![Ruff](https://img.shields.io/badge/lint-ruff-D7FF64?logo=ruff&logoColor=black)
![Cost](https://img.shields.io/badge/cost-free%20tier%20only-success)

**8 workspaces · 30+ allowlisted tools · 1 LangGraph agent · 0 paid APIs**

</div>

---

## 🗺️ At a glance

```mermaid
mindmap
  root((✨ OmniFlow AI))
    💬 Assistant
      Chat with Gemini
      Slash commands
      Saved conversations
      Image understanding
    📄 Documents
      Multi-PDF RAG
      Page citations
      Local embeddings
      Optional OCR
    📊 Data Analysis
      CSV / Excel
      Profiling
      Charts
      Trends & forecasts
      Data stories
    🗄️ SQL
      Natural language → SQL
      Read-only, 2 safety layers
      Import CSV into SQLite
    🔎 Research
      Web search
      News & announcements
      Academic papers
      Weather & currency
    🗂️ Planner
      Goal → milestones → tasks
      Status & next steps
    💻 Code
      Search / read / trace
      Security scan
      Patch proposals
    🧠 Memory
      Preferences
      Project facts
    🔌 Integrations
      Image generation
      Multi-tool orchestration
```

---

## 🧩 System architecture

```mermaid
flowchart LR
  U((👤 User)) --> UI

  subgraph UI["🖥️ Streamlit UI · omniflow/ui"]
    direction TB
    P1[💬 Assistant]
    P2[📄 Documents]
    P3[📊 Data]
    P4[🗄️ SQL]
    P5[🗂️ Planner]
    P6[💻 Code]
    P7[🧠 Memory]
    P8[🔌 Integrations]
  end

  UI --> AG

  subgraph AG["🧠 LangGraph Agent · omniflow/agent"]
    direction TB
    G[graph.py] --- V[verification.py]
    G --- AP[approval.py]
  end

  AG <--> LLM[["🤖 Gemini<br/>primary + fallback"]]
  AG --> REG

  subgraph REG["🧰 Tool Registry + Executor · omniflow/tools"]
    direction TB
    T1[allowlist] --> T2[schema check] --> T3[timeout + retry]
  end

  REG --> DOC[(📄 PDF vector index<br/>in memory)]
  REG --> DATA[(📊 pandas datasets<br/>in memory)]
  REG --> SQL[(🗄️ SQLite<br/>read-only)]
  REG --> EXT{{🌐 Free public APIs}}
  REG --> MEM[(🧠 Memory + chats<br/>SQLite on disk)]

  EXT --- E1[DuckDuckGo]
  EXT --- E2[Tavily · optional]
  EXT --- E3[Open-Meteo]
  EXT --- E4[Frankfurter]
  EXT --- E5[OpenAlex · arXiv]
  EXT --- E6[Cloudflare AI · optional]
```

---

## 🔁 How one message flows through the agent

```mermaid
flowchart TD
  S([📨 user message]) --> I[🚪 intake<br/>validate · /commands · secret rule]
  I -->|needs model| P[🧭 planner<br/>1 Gemini call]
  I -->|/search · /weather …| A
  I -->|/help · refused · invalid| F

  P -->|answer directly| F
  P -->|clarify / unsupported| F
  P -->|bad plan| FB[🛟 fallback chat<br/>no tools] --> F
  P -->|tool calls| A[✋ approval gate<br/>pause for side effects]

  A -->|approved| X[⚙️ execute tools<br/>allowlist · schema · timeout]
  A -->|declined| F
  X -->|evidence| SY[✍️ synthesize<br/>cited answer]
  X -->|nothing found| F
  SY --> F[✅ finalize<br/>verify citations · store]
  F --> E([🏁 END])

  classDef model fill:#8E75B2,color:#fff,stroke:#5b4a7a
  classDef safe fill:#2EA44F,color:#fff,stroke:#1b6e33
  class P,SY,FB model
  class A,F safe
```

| 🟣 Purple = Gemini call | 🟢 Green = safety checkpoint | ♾️ Graph is **acyclic** — it cannot loop |
|:---:|:---:|:---:|

### Model calls per turn

| Request | Route | 🤖 Gemini calls |
|---|---|:---:|
| `/help`, empty, secret-request | deterministic | **0** |
| `/search` `/weather` `/news` `/papers` `/docs` | tool (planner skipped) | **1** |
| Normal question | chat | **1** |
| Needs live info | tool → synthesize | **2** |

---

## 📄 Document RAG pipeline

```mermaid
flowchart LR
  U[📤 PDF] --> V{✔️ valid?<br/>type · size · header}
  V -->|no| R[❌ reject]
  V -->|yes| H{🔁 same SHA-256?}
  H -->|yes| SK[⏭️ skip]
  H -->|no| X[📑 extract pages<br/>pypdf + OCR]
  X --> C[✂️ chunk per page]
  C --> E[🧮 embed locally<br/>bge-small · CPU]
  E --> IDX[(🗂️ session index)]

  Q[❓ question] --> IDX
  IDX -->|score ≥ 0.55| A[🤖 cited answer<br/>file.pdf, page N]
  IDX -->|no match| AB[🙅 abstain]
```

## 📊 Data → Insight pipeline

```mermaid
flowchart LR
  F[📁 CSV / XLSX] --> L[📥 load + validate]
  L --> PR[🧾 profile<br/>types · nulls · outliers]
  L --> Q[💬 question]
  Q --> G[🤖 Gemini picks<br/>1 allowlisted op]
  G --> PD[🐼 pandas computes]
  PD --> OUT{output}
  OUT --> T[📋 table]
  OUT --> CH[📈 chart]
  OUT --> ST[📖 story / forecast]
  T --> DL[⬇️ formula-safe CSV]
```

> 🔒 The model **never writes code** — it only chooses from a fixed menu of operations.

## 🗄️ SQL — two independent safety layers

```mermaid
flowchart LR
  NL[💬 question] --> M[🤖 Gemini writes SELECT]
  M --> L1{🛡️ Layer 1<br/>sqlglot parser<br/>1 read-only query?<br/>allowed tables?}
  L1 -->|❌| B1[blocked]
  L1 -->|✔️| L2{🛡️ Layer 2<br/>read-only connection<br/>authorizer · timeout<br/>row limit}
  L2 -->|❌| B2[blocked]
  L2 -->|✔️| RES[📋 results]
```

## ✋ Human-in-the-loop approval

```mermaid
sequenceDiagram
  autonumber
  actor U as 👤 User
  participant A as 🧠 Agent
  participant G as ✋ Approval gate
  participant T as 🧰 Tool
  U->>A: "Generate an image of …"
  A->>G: planned call (side effect)
  G-->>U: approval card + SHA-256 fingerprint
  U->>G: ✅ Approve
  G->>T: run exactly that call
  T-->>A: result
  A-->>U: verified answer
```

---

## 🧰 Tool belt

| Area | Tools | 🔑 Key needed? |
|---|---|:---:|
| 🔎 Web | `web_search` · `tavily_search` | ❌ · optional |
| 📰 Research | `news_search` · `academic_search` | ❌ |
| 🌦️ Weather | `weather_lookup` | ❌ |
| 💱 Currency | `currency_convert` | ❌ |
| 📄 Documents | `document_search` | ❌ |
| 📊 Data | `dataset_profile` · `dataset_analyze` · `dataset_chart` · `dataset_story` · `dataset_trends` · `dataset_forecast` | ❌ |
| 🗄️ SQL | `sql_schema` · `sql_query` | ❌ |
| 🗂️ Planner | `planner_propose_plan` · `planner_project_status` · `planner_next_tasks` · `planner_list_tasks` · `planner_propose_changes` | ❌ |
| 💻 Code | `code_search` · `code_read` · `code_outline` · `code_trace` · `code_security_scan` · `code_propose_patch` | ❌ |
| 🖼️ Images | `image_analyze` · `image_generate` | ❌ · optional |
| 🧠 Memory | `memory_propose` | ❌ |

> The only required key is a **free** Gemini key.

---

## 🛡️ Safety at a glance

| | Control |
|:---:|---|
| 🚫 | Model can call **only registered tools** — unknown names rejected |
| 🔢 | Per turn: **3** tool calls · **8** graph steps · recursion limit **20** |
| ⏱️ | Every tool has a **timeout**; only transient errors retried, **once** |
| 🧪 | Tool output wrapped as **untrusted evidence** (prompt-injection guard) |
| 🔗 | Fake citations, unknown links and images **stripped** before display |
| 🙅 | No evidence → agent **abstains** instead of guessing |
| 🔐 | Each browser session isolated by a **server-generated id** |
| 🗝️ | Secrets only in `.env` as `SecretStr`; **redacted** from logs |

---

## 📁 Project layout

```text
omniflow-AI/
├── app.py                  🚀 Streamlit entry point
├── omniflow/
│   ├── agent/              🧠 LangGraph graph, prompts, approval, verification
│   ├── tools/              🧰 registry, executor and every tool
│   ├── core/               ⚙️ config, LLM factory, errors, logging
│   ├── documents/          📄 PDF extraction, OCR, chunking, embeddings
│   ├── data/               📊 ingest, profile, charts, trends, forecast, story
│   ├── sql/                🗄️ database, guard, importer, sample data
│   ├── research/           🔎 news + academic sources
│   ├── planner/            🗂️ plans, tasks, changes
│   ├── code/               💻 workspace, checks, security, patches
│   ├── memory/             🧠 store, policy, retrieval
│   ├── integrations/       🔌 image generation hub
│   ├── images/             🖼️ validation + EXIF stripping
│   ├── conversations/      💾 saved chat history
│   └── ui/                 🖥️ one module per page
├── tests/                  🧪 offline pytest suite (+ opt-in live tests)
├── docs/                   📚 user guide, status, API keys
└── langgraph_*.py, streamlit_frontend*.py   🎓 original CampusX tutorial scripts
```

---

## 🚀 Quick start (Windows PowerShell)

```mermaid
flowchart LR
  A[1️⃣ venv] --> B[2️⃣ install] --> C[3️⃣ .env key] --> D[4️⃣ diagnose] --> E[5️⃣ run 🎉]
```

```powershell
py -3.12 -m venv .venv                                              # 1
.\.venv\Scripts\python.exe -m pip install -r requirements-dev.txt   # 2
Copy-Item .env.example .env; notepad .env                           # 3  GEMINI_API_KEY=...
.\.venv\Scripts\python.exe -m omniflow.diagnostics --live           # 4
.\.venv\Scripts\python.exe -m streamlit run app.py                  # 5
```

## 🧪 Tests

```powershell
.\.venv\Scripts\python.exe -m pytest                                # offline · no key · no network
.\.venv\Scripts\python.exe -m ruff check omniflow tests app.py      # lint
$env:OMNIFLOW_RUN_LIVE_TESTS = "1"; .\.venv\Scripts\python.exe -m pytest -m live   # opt-in live
```

---

## 🛠️ Tech stack

| Layer | Tech |
|---|---|
| 🧠 Orchestration | LangGraph · LangChain Core · SQLite checkpoints |
| 🤖 LLM | Google Gemini (with automatic fallback model) |
| 🖥️ UI | Streamlit · Altair (Vega-Lite) |
| 📄 RAG | pypdf · fastembed (ONNX, local) · NumPy · Tesseract *(optional)* |
| 📊 Data | pandas · openpyxl · defusedxml |
| 🗄️ SQL | SQLite · sqlglot |
| 🖼️ Images | Pillow · Cloudflare Workers AI *(optional)* |
| ✅ Quality | pytest · ruff |

---

## 📚 Learn more

| 📖 Doc | What's inside |
|---|---|
| [docs/USER_GUIDE.md](docs/USER_GUIDE.md) | Step-by-step usage of every page, limits, security details, known limitations |
| [docs/STATUS.md](docs/STATUS.md) | What is verified, and how (per phase) |
| [docs/API_KEYS_AND_SERVICES.md](docs/API_KEYS_AND_SERVICES.md) | Keys, free tiers and external services |

<div align="center">

Built on the [CampusX LangGraph chatbot tutorial](https://github.com/campusx-official/chatbot-in-langgraph) · Version **0.6.0**

</div>
