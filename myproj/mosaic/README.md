# MOSAIC
### Multi-Agent Clinical Trial Intelligence System

**An intelligent research integrity analysis system powered by multi-agent orchestration, persistent memory, and clinical research data.**

MOSAIC is a multi-agent AI system designed to analyze clinical trial records and associated research publications to identify potential research integrity concerns, inconsistencies, and overlooked signals.

Built using Python, LangGraph, FastAPI, PostgreSQL, pgvector, and Google Cloud Platform, MOSAIC coordinates six specialized AI agents that independently investigate different dimensions of clinical research integrity and collectively produce an evidence-backed analytical report.

Rather than relying on a single LLM to perform an entire investigation, MOSAIC distributes analytical responsibilities across specialized agents, gives them access to shared research data and persistent memory, and introduces a human feedback mechanism that improves future analysis.

---

## Table of Contents

1. [The Problem](#01-the-problem)
2. [What MOSAIC Does](#02-what-mosaic-does)
3. [Multi-Agent Intelligence](#03-multi-agent-intelligence)
4. [Memory Architecture](#04-memory-architecture)
5. [Human-in-the-Loop Intelligence](#05-human-in-the-loop-intelligence)
6. [Research Data Engineering Pipeline](#06-research-data-engineering-pipeline)
7. [Technology Stack](#07-technology-stack)
8. [Backend API](#08-backend-api)
9. [Project Structure](#09-project-structure)
10. [Engineering Decisions](#10-engineering-decisions)
11. [Current Capabilities and Future Direction](#11-current-capabilities-and-future-direction)
12. [Important Note](#12-important-note)
13. [Project Information](#13-project-information)

---

## 01. The Problem

Clinical research generates an enormous amount of structured and unstructured information.

Trial registrations, study protocols, recruitment timelines, published results, adverse events, and sponsor histories are distributed across multiple research databases.

Individually, these records provide useful information. However, understanding the relationships between them requires connecting information across studies, publications, sponsors, and timelines.

Some important questions include:

- Were the results of a completed clinical trial ever published?
- Did the reported primary outcomes change between registration and publication?
- Are there unexplained delays between expected and actual study completion?
- Does a sponsor have a recurring history of incomplete reporting?
- Are there discrepancies between safety information in trial records and published research?

These questions are difficult to investigate manually at scale.

**MOSAIC approaches this as a multi-agent intelligence problem rather than a conventional document summarization task.**

Its objective is to connect fragmented research information, investigate potential inconsistencies, and surface findings that deserve further human examination.

## 02. What MOSAIC Does

At its core, MOSAIC combines five capabilities:

| Capability | Implementation |
|---|---|
| Research Data Ingestion | ClinicalTrials.gov API v2 and PubMed eUtils |
| Knowledge Retrieval | Embeddings, PostgreSQL and pgvector |
| Distributed Reasoning | Six specialized LangGraph agents |
| Persistent Learning | Episodic, procedural and semantic memory |
| Human Oversight | Confidence-based review and feedback loop |

The system follows a structured workflow:

1. Retrieves clinical trial records and associated research publications.
2. Parses and normalizes the retrieved information.
3. Converts relevant research content into searchable vector representations.
4. Retrieves contextual information for the analysis task.
5. Activates specialized agents to investigate different research integrity dimensions.
6. Aggregates their findings into a consolidated analytical brief.
7. Routes findings through a confidence-based human review mechanism.
8. Stores relevant findings and feedback for future analysis.

The result is a system that does more than generate answers: it maintains knowledge across analysis sessions and incorporates human corrections into its future reasoning process.

---

## 03. Multi-Agent Intelligence

MOSAIC uses **LangGraph** to coordinate six specialized agents through a shared state and a supervisor-controlled workflow.

Each agent has a clearly defined analytical responsibility, its own detection logic, and access to relevant research retrieval and memory tools.

### The Six Specialist Agents

| Agent | Responsibility | Analytical Focus |
|---|---|---|
| Missing Results Agent | Identifies completed trials with missing results | Results reporting obligations and publication gaps |
| Broken Promises Agent | Investigates potential outcome switching | Differences between registered and reported study outcomes |
| Track Record Agent | Evaluates sponsor reporting history | Historical sponsor credibility and reporting patterns |
| Pattern Finder Agent | Identifies relationships across studies | Recurring patterns and cross-study anomalies |
| Side Effect Checker | Examines safety reporting discrepancies | Differences between trial records and published safety information |
| Timeline Analyst | Investigates unexplained study delays | Recruitment, completion, and reporting timelines |

### Why Six Agents?

A single general-purpose LLM would need to perform multiple analytical tasks within one reasoning process.

This creates several challenges:

- Different detection tasks require different contextual information.
- A general-purpose prompt can overlook specialized analytical details.
- Findings from one investigation may interfere with another.
- It becomes difficult to independently evaluate the reliability of individual findings.

MOSAIC addresses this through task specialization.

Each agent investigates its assigned dimension independently, allowing the system to perform multiple investigations concurrently.

### Supervisor

The supervisor coordinates the overall analysis lifecycle.

Its responsibilities include:

- Interpreting the incoming analytical task.
- Determining which specialist agents should participate.
- Managing the shared graph state.
- Coordinating parallel execution.
- Collecting agent outputs.
- Consolidating findings into a final analytical brief.
- Passing results to the human-in-the-loop evaluation layer.

LangGraph provides the orchestration infrastructure through nodes, edges, conditional routing, and shared state.

This makes the workflow modular: individual agents can be modified, evaluated, or extended without redesigning the entire system.

---

## 04. Memory Architecture: Building an AI System That Learns

One of the central design decisions in MOSAIC is that agent intelligence should not be restricted to a single execution session.

The system implements three distinct memory categories using LangMem and persistent storage.

### 4.1 Episodic Memory — Remembering Previous Investigations

Episodic memory stores information about previous analysis sessions.

It allows agents to retrieve relevant historical investigations instead of treating every incoming task as an isolated interaction.

Examples:

- Previously identified reporting gaps.
- Earlier investigations involving the same sponsor.
- Historical findings related to a particular research topic.
- Context from previous analytical sessions.

Relevant memories can be retrieved through semantic search, providing historical context to subsequent investigations.

### 4.2 Procedural Memory — Learning From Human Corrections

Procedural memory stores reasoning instructions and analytical rules that influence how agents approach future tasks.

This is particularly important for human feedback.

When an analyst rejects a finding and provides a reason, that feedback can be translated into an updated reasoning instruction for the relevant agent.

For example:

An agent identifies a completed trial as having potentially missing results.

A human reviewer discovers that the trial was terminated early and that the reporting requirement was interpreted incorrectly.

The correction is incorporated into procedural memory so that future investigations account for this distinction.

The objective is to make human feedback reusable rather than treating it as a one-time correction.

### 4.3 Semantic Memory — Accumulating Sponsor Knowledge

Semantic memory maintains a growing knowledge base of sponsor-related information.

It supports the accumulation and retrieval of sponsor profiles, historical findings, and relevant contextual information.

Over multiple analysis sessions, this allows MOSAIC to build a broader understanding of recurring sponsor-level patterns.

### How the Three Memories Work Together

| Memory | What it stores | Primary purpose |
|---|---|---|
| Episodic | Previous investigations | Historical context |
| Procedural | Reasoning instructions and corrections | Improve future decisions |
| Semantic | Sponsor and domain knowledge | Accumulate contextual knowledge |

Together, these components provide a foundation for persistent, feedback-driven agent behaviour.

---

## 05. Human-in-the-Loop Intelligence

MOSAIC is designed around the principle that AI-generated research integrity signals should not automatically be treated as established misconduct.

Clinical research involves regulatory, methodological, and contextual complexities. A potential discrepancy needs to be examined before being interpreted as an actual integrity concern.

For this reason, MOSAIC incorporates a confidence-based human review mechanism.

### Review Workflow

**High-confidence findings**

- Processed through the normal signal persistence workflow.
- Stored for subsequent retrieval and analysis.

**Low-confidence findings**

- Routed to a dedicated human review queue.
- Presented for analyst evaluation.
- Await an approval, rejection, or correction decision.

### Feedback and Learning Loop

When a reviewer rejects or corrects a finding:

1. The reviewer submits a decision and supporting explanation.
2. The feedback is associated with the relevant agent.
3. The procedural memory layer incorporates the correction into its reasoning instructions.
4. Future investigations can retrieve and apply the updated knowledge.

This creates a feedback-driven architecture in which human expertise becomes part of the system's persistent knowledge.

**The goal is not to eliminate human judgment, but to make it an integral part of the intelligence pipeline.**

---

## 06. Research Data Engineering Pipeline

MOSAIC integrates two major public research information sources.

### Data Sources

**ClinicalTrials.gov API v2**

Provides structured clinical trial information, including:

- Trial identification and registration.
- Study status.
- Sponsor information.
- Study design.
- Primary and secondary outcomes.
- Recruitment information.
- Study start and completion dates.
- Results availability.

**PubMed eUtils**

Provides access to biomedical research publications and associated metadata.

The integration supports:

- Publication searches.
- Article retrieval.
- Research metadata extraction.
- Linking relevant publications with clinical trial investigations.

### Data Ingestion

The ingestion layer is responsible for retrieving and preparing information from external research sources.

Key components:

- `clinical_trials_client.py` — communicates with ClinicalTrials.gov.
- `pubmed_client.py` — performs PubMed searches and article retrieval.
- `document_parser.py` — converts retrieved information into structured application models.
- `gcs_store.py` — persists raw and processed research data in Google Cloud Storage.

The clients use asynchronous HTTP communication where appropriate, together with retry handling and rate-limit-aware request management.

### Data Processing

Retrieved research information is transformed into a format suitable for semantic retrieval.

**1. Chunking**

Documents are divided into manageable text segments using approximately 500-word chunks with 50-word overlap.

This preserves relevant context across chunk boundaries while making retrieval more targeted.

**2. Embedding Generation**

Text segments are converted into numerical vector representations using OpenAI's `text-embedding-3-small` embedding model.

Each embedding contains 1536 dimensions.

**3. Vector Persistence**

Embeddings and their associated metadata are stored in PostgreSQL using pgvector.

**4. Semantic Retrieval**

The system uses vector similarity search to retrieve relevant research context for downstream agents.

An HNSW index supports efficient approximate nearest-neighbour search.

### Pipeline Overview

```text
ClinicalTrials.gov + PubMed
            |
            v
API Clients and Data Retrieval
            |
            v
Parsing and Data Normalization
            |
            v
Document Chunking
            |
            v
Embedding Generation
            |
            v
PostgreSQL + pgvector
            |
            v
Agent Retrieval Tools
            |
            v
Specialized Analysis
```

---

## 07. Technology Stack

MOSAIC combines agent orchestration, LLM-based reasoning, semantic retrieval, asynchronous backend development, and managed cloud infrastructure.

### Core Development

| Technology | Role in MOSAIC |
|---|---|
| Python 3.12 | Primary application and orchestration language |
| Asyncio | Asynchronous execution and resource coordination |
| Pydantic v2 | Data validation, structured models, and configuration |
| Tenacity | Retry handling for external service communication |

### AI and Agent Orchestration

| Technology | Role in MOSAIC |
|---|---|
| LangGraph 0.2.6 | Multi-agent workflow orchestration and parallel execution |
| LangMem | Persistent agent memory and feedback-driven learning |
| GPT-4o | Language reasoning, contextual analysis, and report generation |
| OpenAI Embeddings | Semantic representation of research documents |
| text-embedding-3-small | 1536-dimensional document embeddings |

### Backend and API

| Technology | Role in MOSAIC |
|---|---|
| FastAPI | REST API and application interface |
| Uvicorn | ASGI application server |
| Pydantic v2 | Request and response schema validation |
| HTTPX | Asynchronous external API communication |
| Requests | HTTP communication with supported external services |

### Database and Retrieval

| Technology | Role in MOSAIC |
|---|---|
| PostgreSQL 15 | Persistent relational storage |
| pgvector | Vector storage and semantic similarity search |
| HNSW | Approximate nearest-neighbour indexing |
| asyncpg | Asynchronous PostgreSQL connectivity |

### Google Cloud Infrastructure

| Technology | Role in MOSAIC |
|---|---|
| Google Cloud Run | Serverless application hosting |
| Cloud SQL | Managed PostgreSQL database |
| Google Cloud Storage | Research document and artifact storage |
| Secret Manager | Secure application secret management |
| Docker | Containerized application packaging |

### External Research Integrations

| Service | Purpose |
|---|---|
| ClinicalTrials.gov API v2 | Clinical trial registrations and study information |
| PubMed eUtils | Biomedical literature retrieval |
| Google Cloud APIs | Managed infrastructure integration |
| OpenAI API | LLM reasoning and embedding generation |

---

## 08. Backend API

MOSAIC exposes its analytical capabilities through a REST API built using FastAPI.

The API separates analytical execution, signal retrieval, human review, and memory inspection into dedicated routes.

### API Endpoints

| Method | Endpoint | Responsibility |
|---|---|---|
| POST | `/api/v1/analyze` | Initiates a multi-agent analysis |
| GET | `/api/v1/signals` | Retrieves generated signals |
| GET | `/api/v1/signals/{id}` | Retrieves a specific signal |
| GET | `/api/v1/review/queue` | Retrieves pending human reviews |
| PATCH | `/api/v1/review/{id}` | Submits a human review decision |
| GET | `/api/v1/memory/episodes` | Searches episodic memory |
| GET | `/api/v1/memory/procedures/{agent}` | Retrieves agent reasoning rules |
| GET | `/api/v1/sponsors` | Retrieves sponsor profiles |
| GET | `/api/v1/sponsors/{name}` | Retrieves a specific sponsor profile |
| GET | `/api/v1/health` | Checks application health |

### API Design Principles

- Separation of concerns through dedicated routers.
- Pydantic-based request and response validation.
- Shared resource initialization through dependency management.
- Centralized application configuration.
- Consistent API versioning.
- Structured error handling and logging.

Interactive API documentation is generated through FastAPI's OpenAPI integration.

---

## 09. Project Structure

```text
mosaic/
│
├── config/
│   ├── settings.py
│   └── logging_config.py
│
├── ingestion/
│   ├── clinical_trials_client.py
│   ├── pubmed_client.py
│   ├── document_parser.py
│   ├── gcs_store.py
│   └── run_ingestion.py
│
├── processing/
│   ├── chunker.py
│   ├── embedder.py
│   ├── vector_store.py
│   └── run_processing.py
│
├── memory/
│   ├── episodic_store.py
│   ├── procedural_store.py
│   └── semantic_store.py
│
├── agents/
│   ├── supervisor.py
│   ├── missing_results_agent.py
│   ├── broken_promises_agent.py
│   ├── track_record_agent.py
│   ├── pattern_finder_agent.py
│   ├── side_effect_agent.py
│   └── timeline_agent.py
│
├── tools/
│   ├── search_tools.py
│   ├── clinical_tools.py
│   └── pubmed_tools.py
│
├── graph/
│   ├── state.py
│   ├── graph_builder.py
│   └── hitl.py
│
├── api/
│   ├── main.py
│   ├── schemas.py
│   ├── dependencies.py
│   └── routers/
│       ├── analysis.py
│       ├── signals.py
│       ├── review.py
│       └── memory.py
│
├── deployment/
│   ├── Dockerfile
│   └── gcp/
│       ├── cloudsql_init.sql
│       └── deploy.sh
│
├── pyproject.toml
├── requirements.txt
├── .env.example
└── README.md
```

The codebase follows a modular architecture so that data ingestion, document processing, agent reasoning, memory management, and API functionality remain independently maintainable.

---

## 10. Engineering Decisions

Some of the important architectural decisions behind MOSAIC:

**Why LangGraph instead of a simple sequential LLM chain?**

The problem requires multiple independent investigations, conditional execution, shared state, and a feedback mechanism. LangGraph provides the graph-based orchestration required to coordinate these operations.

**Why PostgreSQL with pgvector?**

MOSAIC needs both conventional relational persistence and semantic retrieval. Using PostgreSQL with pgvector allows structured application data and vector representations to coexist within the same database infrastructure.

**Why three types of memory?**

Previous investigations, corrections to reasoning, and accumulated sponsor knowledge represent different kinds of information. Separating them makes memory retrieval and updates more intentional.

**Why human-in-the-loop evaluation?**

A research discrepancy does not automatically establish misconduct. Human evaluation helps distinguish genuine reporting concerns from contextual exceptions and improves the quality of subsequent analysis.

**Why FastAPI?**

The analytical engine and API layer have different responsibilities. FastAPI exposes the underlying system through a structured interface without tightly coupling external consumers to internal agent implementations.

---

## 11. Current Capabilities and Future Direction

MOSAIC establishes the foundation for an extensible clinical research intelligence platform.

Potential extensions include:

- **Evaluation Framework:** Introduce systematic agent evaluation using LangSmith and task-specific evaluation datasets.
- **Research Integrity Dashboard:** Develop a dedicated interface for reviewing signals, sponsor histories, and agent findings.
- **Additional Data Sources:** Integrate FDA adverse event data and international clinical trial registries.
- **Scheduled Intelligence:** Introduce recurring analysis through Cloud Scheduler.
- **Alerting System:** Notify analysts when important research integrity signals are generated.
- **Sponsor Intelligence:** Expand historical sponsor analysis into comparative reporting and risk assessment.
- **Improved Retrieval:** Evaluate domain-specific embeddings and more advanced retrieval strategies.

---

## 12. Important Note

MOSAIC is an AI-assisted research intelligence and screening system.

Its outputs represent potential research integrity signals, not definitive findings of research misconduct, regulatory violations, or sponsor wrongdoing.

All findings require appropriate contextual interpretation and, where necessary, independent human verification.

The system is intended to support research oversight and investigation, not replace scientific, regulatory, or clinical judgment.

---

## 13. Project Information

| Attribute | Details |
|---|---|
| Project | MOSAIC — Multi-Agent Clinical Trial Intelligence System |
| Core Approach | Multi-agent AI with persistent memory and human feedback |
| Backend | Python, FastAPI |
| Agent Orchestration | LangGraph |
| LLM | GPT-4o |
| Database | PostgreSQL + pgvector |
| Cloud Infrastructure | Google Cloud Platform |
| Data Sources | ClinicalTrials.gov and PubMed |
| Deployment | Google Cloud Run |
| License | MIT |

---

*Built to explore how specialized AI agents, persistent memory, and human expertise can work together to make complex clinical research information more accessible and actionable.*