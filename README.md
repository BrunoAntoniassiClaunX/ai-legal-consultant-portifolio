# ⚖️ Legal Research Assistant — AI for Law Firms

> AI agent for Brazilian law firms: document ingestion, clause extraction, precedent search via vector similarity, and automated legal report generation.

[![Status](https://img.shields.io/badge/status-portfolio-blue?style=flat)]()
[![Stack](https://img.shields.io/badge/stack-n8n%20·%20GPT--4o%20·%20Pinecone%20·%20PDF%20parsing-0A0A0F?style=flat)]()
[![Output](https://img.shields.io/badge/output-DOCX%20reports-2B579A?style=flat&logo=microsoftword&logoColor=white)]()

---

## 📌 Overview

Legal research is one of the most time-consuming tasks in any law firm — junior associates spend hours reading through case files, identifying relevant clauses, and cross-referencing precedents. This agent automates the entire research pipeline, from raw document ingestion to structured report delivery.

**The business problem:**
- A single litigation case can involve hundreds of documents
- Finding relevant precedents manually takes days per case
- Junior associate time costs R$150–300/h and is largely spent on search, not reasoning
- Contract review for clause extraction is repetitive and error-prone

**What the agent does:**
- Ingests case documents (PDF, DOCX) and indexes them into a vector database
- Extracts specific clause types on demand (liability, termination, penalty, jurisdiction)
- Searches for precedents by semantic similarity across the indexed case library
- Generates structured legal research reports in DOCX format
- Answers natural language questions about any indexed document
- Flags potentially problematic clauses based on configurable risk patterns

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    DOCUMENT INGESTION                        │
│                                                             │
│  Upload PDF/DOCX (via webhook or folder watch)              │
│          │                                                  │
│          ▼                                                  │
│   n8n Cloud                                                 │
│          │                                                  │
│          ├─► PDF text extraction (PyMuPDF / pdfplumber)     │
│          │                                                  │
│          ├─► Chunking (512 tokens, 50 token overlap)        │
│          │                                                  │
│          ├─► OpenAI Embeddings (text-embedding-3-small)     │
│          │                                                  │
│          └─► Pinecone Upsert (with doc metadata)            │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                     RESEARCH QUERY                          │
│                                                             │
│  Lawyer submits query (WhatsApp / web form / email)         │
│          │                                                  │
│          ▼                                                  │
│   n8n Cloud                                                 │
│          │                                                  │
│          ├─► Embed query → Pinecone similarity search       │
│          │   (top_k = 10, filtered by case/doc metadata)    │
│          │                                                  │
│          ├─► GPT-4o (synthesize findings + extract clauses) │
│          │                                                  │
│          ├─► Structure output (sections, citations, risk)   │
│          │                                                  │
│          └─► Generate DOCX report → deliver via email/WA   │
└─────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Orchestration | **n8n Cloud** | Workflow engine, document routing, scheduling |
| LLM | **OpenAI GPT-4o** | Clause extraction, synthesis, report generation |
| Vector Search | **Pinecone** | Semantic search over case documents and precedents |
| PDF Parsing | **pdfplumber / PyMuPDF** | Text extraction from legal documents |
| Report Output | **DOCX generation** | Structured legal reports via python-docx |
| Delivery | **Whapi / Gmail** | Report delivery via WhatsApp or email |

---

## 🔄 n8n Workflow Structure

### Document Ingestion Pipeline

```
Webhook (file upload) OR Schedule Trigger (folder watch)
  └─► File type detection (PDF / DOCX)
        └─► Text extraction
              └─► Chunking (512 tokens, 50 overlap)
                    └─► Metadata tagging
                          │  case_id, doc_type, date, parties
                          └─► OpenAI Embeddings
                                └─► Pinecone Upsert
```

### Research Query Pipeline

```
Inbound query (natural language)
  └─► Intent classification
        ├─► "find precedents" → similarity search → GPT synthesis
        ├─► "extract clauses" → targeted search → structured extraction
        ├─► "review contract" → full doc analysis → risk flagging
        └─► "answer question" → RAG → direct response
              └─► DOCX report generation
                    └─► Delivery (email / WhatsApp)
```

---

## 📄 Clause Extraction

The agent can extract and classify the following clause types:

```
CONTRACT ANALYSIS
─────────────────
Parties         →  Identifies all contracting parties and their roles
Obligations     →  What each party must do
Deadlines       →  All dates and timeframes mentioned
Penalty clauses →  Financial penalties, breach consequences
Termination     →  Conditions for contract termination
Jurisdiction    →  Which court/law governs disputes
Confidentiality →  NDA and information protection clauses
Liability caps  →  Limitation of liability provisions
```

### Risk Flagging

Clauses are scored by risk level:

```json
{
  "clause": "The contracted party assumes unlimited liability...",
  "type": "liability",
  "risk_level": "HIGH",
  "reason": "No liability cap defined. Exposes client to unlimited financial risk.",
  "recommendation": "Negotiate a liability cap of 12x monthly contract value."
}
```

---

## 📊 Report Output Structure

Generated DOCX reports follow this structure:

```
1. Executive Summary
   └─► Key findings, risk overview, recommended actions

2. Document Analysis
   └─► Per-document summary with extracted clauses

3. Precedent Research
   └─► Semantically similar cases with relevance scores and citations

4. Risk Assessment
   └─► Flagged clauses ranked by risk level (HIGH / MEDIUM / LOW)

5. Recommendations
   └─► GPT-4o synthesized legal guidance based on findings

6. Appendix
   └─► Source document references, chunk citations, confidence scores
```

---

## 🧠 Prompt Engineering Notes

Legal prompts require specific design patterns to ensure accuracy:

- **Role definition** — "You are a senior Brazilian legal researcher..."
- **Output schema** — strict JSON structure for clause extraction to enable DOCX templating
- **Hallucination prevention** — "Only cite information present in the provided context. If the answer is not in the documents, say so explicitly."
- **Citation enforcement** — every claim must reference the source chunk (doc name + page)
- **Language** — Brazilian Portuguese legal register, formal (not conversational)

---

## 📈 Expected Results

| Metric | Manual Process | With Agent |
|---|---|---|
| Contract review time (50 pages) | 4–6 hours | 8–12 minutes |
| Precedent search | 1–2 days | 2–5 minutes |
| Associate time per case | 60–80% on research | < 20% on research |
| Consistency of clause extraction | Variable | Standardized |
| Report generation | Manual, hours | Automated, minutes |

---

## 🗂️ Repository Structure

```
legal-ai-assistant/
├── workflows/
│   ├── document_ingestion.json       # PDF/DOCX ingestion pipeline
│   ├── research_query.json           # Research and extraction workflow
│   └── report_delivery.json          # DOCX generation + delivery
├── prompts/
│   ├── research_agent.md             # Main research agent system prompt
│   ├── clause_extraction.md          # Structured clause extraction prompt
│   └── risk_assessment.md            # Risk flagging prompt
├── templates/
│   └── legal_report_template.docx    # DOCX report template
├── docs/
│   ├── architecture.md
│   ├── pinecone_setup.md
│   └── supported_clause_types.md
└── README.md
```

---

## ⚠️ Important Disclaimer

> This tool is designed to **assist** legal professionals, not replace them. All AI-generated research, clause extractions, and risk assessments must be reviewed and validated by a qualified lawyer before being used in any legal proceeding or contract negotiation. The agent does not provide legal advice.

---

## 🏢 About ClaunX

**ClaunX Automation & AI** — *Unlocking Intelligence. Prosperity Through Automation.*

[claunx.com](https://claunx.com) · [LinkedIn](https://linkedin.com/in/brunoantoniassi)

---

*Built by [Bruno Antoniassi](https://github.com/BrunoAntoniassiClaunX) · ClaunX Automation & AI · São Paulo, Brazil*
