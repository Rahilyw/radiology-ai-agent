# Radiology AI Agent

![Python](https://img.shields.io/badge/Python-3.11+-3776AB?logo=python&logoColor=white)
![MCP](https://img.shields.io/badge/Protocol-MCP-FF6B35)
![MedImageParse](https://img.shields.io/badge/MedImageParse-3D-0078D4?logo=microsoft-azure&logoColor=white)
![GitHub Copilot](https://img.shields.io/badge/GitHub_Copilot-Agent-000000?logo=github&logoColor=white)

A radiology AI agent that takes a brain MRI from a bare LLM draft to a **measured, context-aware report** — segmenting lesions with a medical foundation model, turning masks into millimeter measurements in code, and grounding the write-up in synthetic patient history.

Built at **Microsoft Ignite 2026** (LAB730: Build Healthcare Agents with Foundation and Reasoning Models).

> Educational / research exploration only — **not for clinical use**. Sample scans and patient charts are synthetic or de-identified.

---

## The Problem It Solves

A general-purpose LLM can draft a radiology-style report, but it cannot reliably:

- **See** which tumor components are present in a 3D volume
- **Measure** lesion size in real millimeters from voxel geometry
- **Compare** interval change against a prior scan with deterministic numbers
- **Ground** findings in the patient's chart instead of inventing context

This project shows how to fix that by splitting responsibility across layers: a foundation model for segmentation, Python for measurement, MCP tools for the agent, and skills for structured report writing.

---

## How It Works

```
┌──────────────────────────────────────────────────────────────────┐
│                     Radiology Assistant (Copilot agent)          │
│                                                                  │
│  Skills: write-report-measured · write-imaging-factsheet         │
│          lookup-patient-history · write-report-with-ehr          │
│                                                                  │
│  ┌─────────────────────┐   ┌──────────────────────────────────┐  │
│  │  MCP: MedImageParse │   │  Deterministic measurement code  │  │
│  │  list_models        │──▶│  clean_segmentation              │  │
│  │  segment (3D MRI)   │   │  describe_lesions (mm, mL, Δ)    │  │
│  └─────────────────────┘   └──────────────────────────────────┘  │
│                                                                  │
│  + Synthetic EHR / prior report for interval-change context      │
│  + Per-image audit-log/ (masks, montages, descriptors, drafts)   │
└──────────────────────────────────────────────────────────────────┘
```

### 1 — Simple agent + skill
Start with a prompt-based skill that drafts a structured brain MRI report from a general-purpose LLM. Useful for structure — and clear about where guessing begins.

### 2 — Foundation model via MCP
Wrap **MedImageParse 3D** (text-prompted whole-volume segmentation) in an MCP server. The agent calls `segment`, then measurement tools that clean the mask and emit lesion IDs, volumes, and longest diameters in millimeters.

### 3 — Patient context
Ground the draft in a synthetic patient history and prior report so the agent can speak to interval change instead of a single snapshot in isolation.

**Design rule:** models return masks; code returns facts; the LLM writes words.

---

## What I Built / Completed

| Area | Outcome |
|------|---------|
| Agent | Copilot radiology assistant wired to MCP tools and report skills |
| MCP server | `medimageparse` tools: catalog, segment, clean, describe |
| Measurement | Voxel-spacing-aware volumes and diameters; prior-scan delta |
| Knowledge | Synthetic patient charts + prior reports for grounded drafting |
| Cases | De-identified / synthetic post-contrast brain MRI volumes (NIfTI) |

---

## Project Structure

```
.
├── .github/
│   ├── agents/radiology-assistant.agent.md   # Agent persona + workflow
│   └── skills/                               # Report, factsheet, EHR skills
├── mcp/medimageparse-server/                 # MCP server + segmentation/lesion code
├── chapters/                                 # Progressive build (starter + reference)
├── data/                                     # Synthetic patients, reports, images
├── samples/mri-brain-lesions/                # Case folders for demos
├── workshop/                                 # Env check + shared helpers
├── docs/                                     # Setup / Codespaces / troubleshooting
└── pyproject.toml
```

---

## Setup

### Prerequisites
- VS Code + GitHub Copilot Chat
- Git LFS (`git lfs install` before clone)
- [uv](https://docs.astral.sh/uv/)
- MedImageParse endpoint credentials (`MIP_MODEL_ENDPOINT`, `MIP_MODEL_API_KEY`)

### Install

```bash
git lfs install
git clone https://github.com/Rahilyw/radiology-ai-agent.git
cd radiology-ai-agent
uv sync
cp .env.example .env   # fill in MedImageParse credentials
uv run workshop/check_env.py
```

### Run the MCP server

```bash
uv run --directory mcp/medimageparse-server mcp run server.py
```

Or use the workspace `.vscode/mcp.json` registration and talk to the agent in Copilot Chat.

Copy a completed chapter into the workspace root when you want the full agent + skills layout:

```bash
cp -r chapters/chapter2-foundation-models/answer-key/. .
# or chapter3 for EHR-grounded reporting
```

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| Agent host | GitHub Copilot agent + skills |
| Tool protocol | MCP (Model Context Protocol) |
| Foundation model | MedImageParse 3D (MRI brain lesion segmentation) |
| Measurement | Python (NIfTI geometry, connected components) |
| Packaging | uv / Python 3.11+ |
| Samples | NIfTI volumes via Git LFS |

---

## Attribution

Workshop materials and MedImageParse integration patterns originate from Microsoft’s Ignite 2026 LAB730 (`microsoft/LAB730-ignite-2026-healthcare-agents`), licensed under MIT. This repository is my portfolio build of that lab — completed agent wiring, synthetic case data, and documentation rewritten for personal presentation.

---

*Built at Microsoft Ignite 2026 — LAB730: Build Healthcare Agents with Foundation and Reasoning Models*
