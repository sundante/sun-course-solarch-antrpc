# Apic Applied AI Architect — Interview Prep

**Live Site (Coming Soon):** https://learnapic.sunmintz.com

A **comprehensive, depth-first curriculum** for interview preparation in the **Apic Applied AI Architect** role (Mumbai, India/APAC). Combines structured prose notes, graded practice drills, production system designs, and interview Q&A banks across 12 modules.

---

## 📚 Curriculum Structure

### 12 Modules | 60+ Concept Notes | 36 Graded Drills | 260+ Interview Q&A | 11 Code Labs

| # | Module | Focus | Status |
|---|--------|-------|--------|
| 01 | **Apic Calibration** | Mission, role, culture, interview loop | ✅ Complete |
| 02 | **Personal Narrative & Role-Fit** | Career arc, lateral move, mission alignment | 🔄 WIP |
| 03 | **Customer Discovery & Pre-Sales** | Discovery process, stakeholder mapping, AE partnership | 🔄 WIP |
| 04 | **Claude Platform Architecture** | API surface, model selection, prompt caching, tool use | 🔄 WIP |
| 05 | **Solution Architecture Patterns** | Reference architecture, integration doctrine, failure modes | 🔄 WIP |
| 06 | **Evaluation Architecture** | Eval design, quality gates, metric selection | 🔄 WIP |
| 07 | **RAG in Regulated Enterprises** | Retrieval patterns, RBAC, data governance, RBI compliance | 🔄 WIP |
| 08 | **Agentic & Tool-Use Architecture** | Multi-agent patterns, orchestration, error handling | 🔄 WIP |
| 09 | **Security, Privacy & Governance** | DPDPA 2023, data residency, audit, CISO collaboration | 🔄 WIP |
| 10 | **Cost, Latency & Reliability** | Resource optimization, SLA design, failure analysis | 🔄 WIP |
| 11 | **Storytelling, Workshops & Demos** | Demo design, technical storytelling, customer comms | 🔄 WIP |
| 12 | **Full-Loop Simulation** | End-to-end deployment, production readiness | 🔄 WIP |

---

## 🗂️ Repository Architecture

```
repository/
├── docs/                           ← MkDocs-served curriculum
│   ├── 01-Apic-Calibration/   ← Each module follows design pattern
│   │   ├── INDEX.md                ← Module overview & navigation
│   │   ├── Notes/                  ← Numbered concept files (01-, 02-, ...)
│   │   ├── Drills/                 ← Graded practice exercises
│   │   ├── Case-Study/             ← Running BFSI customer scenario
│   │   ├── CodeLabs/               ← Executable notebooks (if applicable)
│   │   └── SystemDesigns/          ← Architecture patterns (if applicable)
│   ├── ... (02-12 modules)
│   ├── Interview-Questions/        ← Cross-module Q&A aggregator
│   ├── Drill-Tracker.md            ← Central graded drill log
│   └── stylesheets/
│
├── Codes/                          ← Runnable labs & implementations
│   ├── setup/                      ← One-time configuration
│   │   ├── requirements.txt         ← Dependencies
│   │   ├── .env.example             ← Config template
│   │   └── setup-guide.md           ← Getting started
│   ├── 01-Concepts/                ← Single concept explorations
│   ├── 02-Architectures/           ← Pattern implementations
│   └── 03-End-to-End-Systems/      ← Full application examples
│
├── Resources/                      ← Reference materials
│   └── (PDFs, papers, external links)
│
├── vibes/                          ← Meta-documentation
│   ├── design.md                   ← This design spec
│   ├── status.md                   ← Content completion tracker
│   ├── handoff.md                  ← Session state & next steps
│   ├── plan.md                     ← 12-module plan
│   └── Build-Plan.md               ← Weekly build schedule
│
├── mkdocs.yml                      ← MkDocs configuration
├── README.md                       ← This file
├── .github/
│   └── workflows/
│       └── deploy.yml              ← GitHub Pages auto-deploy
└── LICENSE, CNAME, .gitignore
```

---

## 🚀 Quick Start

### Read the Curriculum

```bash
# Install dependencies
pip install mkdocs-material

# Start local preview (with live reload)
mkdocs serve
# → http://127.0.0.1:8000
```

### Run Code Labs

```bash
cd Codes
python3 -m venv .venv
source .venv/bin/activate
pip install -r setup/requirements.txt
cp setup/.env.example .env  # Configure API keys

# Run Jupyter notebooks
jupyter notebook
```

See [Codes/setup/setup-guide.md](Codes/setup/setup-guide.md) for details.

### Validate Build

```bash
mkdocs build --strict  # Check for broken links
```

---

## 📖 Module Design: Depth-First Progression

Each module follows a consistent learning path:

| Phase | Content | Depth |
|-------|---------|-------|
| 1️⃣ **Foundations** | Core concepts, mental models, definitions | Beginner |
| 2️⃣ **Building Blocks** | Components, how they interact, patterns | Intermediate |
| 3️⃣ **Architecture** | System design, trade-offs, constraints | Advanced |
| 4️⃣ **Production** | Real-world deployment, failure modes, optimization | Expert |
| 5️⃣ **Interview Q&A** | Consolidated knowledge check across all phases | Assessment |

### Example: Module Progression

```
01-Apic-Calibration/
├── INDEX.md                                ← Start here
├── Notes/
│   ├── 01-Mission-Values-Constitutional-AI.md
│   ├── 02-Role-Decoded-Applied-AI-Architect.md
│   ├── 03-Interview-Loop-Map.md
│   ├── 04-What-Good-vs-Weak-Looks-Like.md
│   └── 05-Interview-QA-Bank.md            ← Final knowledge check
├── Drills/
│   ├── 01-Tell-Me-About-Yourself.md
│   ├── 02-Why-Apic.md
│   └── 03-Customer-Architecture-Story.md
└── Case-Study/
    └── 01-BFSI-Customer-Intro.md
```

---

## 🎯 Learning Path

1. **Start:** Read module `INDEX.md` for overview and learning path
2. **Concept notes:** Work through `Notes/01-*` → `Notes/N-*` sequentially
3. **Practice:** Complete 3 Drills per module with timed, raw answers
4. **Real-world:** Study running BFSI case-study scenarios
5. **Self-check:** Review Interview-QA-Bank for knowledge gaps
6. **Code labs:** Explore implementation patterns in `Codes/` (if applicable)

---

## 🎨 Technology Stack

### Documentation
- **MkDocs Material** — Fast, searchable, GitHub Pages ready
- **Mermaid** — System design diagrams
- **KaTeX** — Mathematical notation
- **Admonitions** — Personal anchors, graded verdicts

### Code Execution
- **Jupyter Notebooks** — Interactive exploration
- **Python** — All scripts and labs
- **Poetry/venv** — Dependency management

### AI Frameworks (in Code Labs)
- **Apic Claude SDK** — Core API interactions
- **LangChain / LangGraph** — Multi-framework patterns
- **CrewAI** — Multi-agent orchestration
- **Google Vertex AI** — Enterprise deployment

---

## 📝 Content Guidelines

### Merge-Friendly Structure

This course will be merged into `sun-course-solarch` (generic AI Solutions Architect). To keep the merge cheap:

- **Generic bodies:** Concept notes are written for *any* Architect candidate
- **Personal anchors:** Candidate-specific stories live in `!!! personal` admonitions
- **Graded verdicts:** Drill rubrics use `!!! grade` blocks

**Merge strategy:** Strip the personal blocks, keep the generic body.

### BFSI Grounding

All examples use **real Indian BFSI context:**
- RBI guidelines & DPDPA 2023
- In-India data residency
- Bedrock + Vertex India regions
- Production system constraints (hallucination tolerance ≈ 0, HITL required)

---

## 🏆 Grading Rubric

All drills are graded on a 4-point scale:

| Grade | Criteria |
|-------|----------|
| 🟢 **Strong Hire** | Specific, credible, mission-aligned, shows depth + architecture maturity |
| 🟡 **Hire** | Mostly right; missing one dimension; fixable with revision |
| 🔴 **Lean No** | Generic, buzzword-heavy, shallow, or performance-based |
| ⚫ **Strong No** | Wrong, dishonest, unsafe judgment, misaligned |

**Application trigger:** Only apply when both "Tell me about yourself" and "Why Apic" are at **Strong Hire** level.

---

## 📊 Tracking Progress

- **[vibes/status.md](vibes/status.md)** — Content completion tracker
- **[docs/Drill-Tracker.md](docs/Drill-Tracker.md)** — Graded drill history
- **[vibes/handoff.md](vibes/handoff.md)** — Current session state

---

## 🔗 Related Projects

- [sun-course-genai](https://github.com/sundante/sun-course-genai) — Public GenAI curriculum (mirror structure)
- [sun-course-solarch](coming-soon) — Generic AI Solutions Architect course (future merge target)

---

## 📄 License

See [LICENSE](LICENSE).

---

## 🛠️ Deployment

GitHub Actions automatically builds and deploys to GitHub Pages on every push to `main`. See [.github/workflows/deploy.yml](.github/workflows/deploy.yml).

**Status:** Not yet deployed. CNAME configured for `learnapic.sunmintz.com`. Enable GitHub Pages in repository settings to activate.
