# Ontology in Microsoft IQ — Technical Documentation

> **Scope:** Microsoft Fabric IQ · Microsoft Foundry IQ  
> **Audience:** Data Engineers · Solution Architects · AI/ML Engineers  
> **Status:** Fabric IQ Ontology — Public Preview (May 2026) · Foundry IQ — Public Preview (GA planned Q2 2026)  
> **Sources:** [Microsoft Learn – Fabric IQ](https://learn.microsoft.com/en-us/fabric/iq/overview) · [Microsoft Learn – Foundry IQ](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/what-is-foundry-iq)

---

## Table of Contents

- [1. What Is an Ontology?](#1-what-is-an-ontology)
- [2. What Is Microsoft IQ?](#2-what-is-microsoft-iq)
- [3. Fabric IQ — The Structured Data Ontology](#3-fabric-iq--the-structured-data-ontology)
  - [3.1 Core Building Blocks](#31-core-building-blocks)
  - [3.2 Entity Types](#32-entity-types)
  - [3.3 Properties](#33-properties)
  - [3.4 Relationship Types](#34-relationship-types)
  - [3.5 Data Bindings](#35-data-bindings)
  - [3.6 How the Ontology Is Created](#36-how-the-ontology-is-created)
  - [3.7 Full Fabric IQ Architecture](#37-full-fabric-iq-architecture)
  - [3.8 End-to-End Example Flow](#38-end-to-end-example-flow)
  - [3.9 Security and Governance](#39-security-and-governance)
- [4. Foundry IQ — The Unstructured Knowledge Layer](#4-foundry-iq--the-unstructured-knowledge-layer)
  - [4.1 The Problem It Solves](#41-the-problem-it-solves)
  - [4.2 Knowledge Base Architecture](#42-knowledge-base-architecture)
  - [4.3 Agentic Retrieval Engine](#43-agentic-retrieval-engine)
  - [4.4 Supported Knowledge Sources](#44-supported-knowledge-sources)
  - [4.5 Integration Code Example](#45-integration-code-example)
  - [4.6 Security and Governance](#46-security-and-governance)
- [5. Fabric IQ vs Foundry IQ — Direct Comparison](#5-fabric-iq-vs-foundry-iq--direct-comparison)
- [6. How They Work Together](#6-how-they-work-together)
- [7. Decision Guide — Which Layer for Which Problem?](#7-decision-guide--which-layer-for-which-problem)
- [References](#references)

---

## 1. What Is an Ontology?

An **ontology** is a formal, machine-readable specification of:

1. **Concepts / Entities** — the things that exist in a domain (Customer, Order, Shipment)
2. **Properties** — the attributes of those things (Customer.email, Shipment.status)
3. **Relationships** — how things connect to each other (Customer *places* Order)
4. **Rules / Constraints** — what is valid, required, or impossible (Order.amount must be > 0)

### What an Ontology Is NOT

| Not This | Why It Falls Short |
|---|---|
| A glossary | Plain text — not queryable, not machine-readable |
| A data dictionary | Describes columns, not business concepts |
| A database schema | Structural, not semantic — no business meaning |
| A taxonomy | Hierarchy only — no relationships or rules |

An ontology is all four of those things **combined**, made **live, queryable, and agent-accessible**.

### Why It Matters for AI

When a user asks Copilot *"What is our revenue this quarter?"*, the AI faces three ambiguous terms:

- **Revenue** — gross or net? which currency? which regions?
- **Our** — which business unit or subsidiary?
- **This quarter** — fiscal or calendar? which fiscal year start?

Without an ontology, the AI **guesses**. The failure mode is silent: a confident answer that is confidently wrong.

With an ontology, every term has one formal definition accessible by both humans and machines.

```
"Better context = better AI.
 The bottleneck was never the model. It was always the vocabulary."
```

---

## 2. What Is Microsoft IQ?

> ⚠️ **Note:** "Foundry IQ" is a Microsoft product — it is **not** Palantir Foundry. Both are Microsoft products from the same IQ family, announced at **Microsoft Ignite 2025**.

Microsoft IQ is a unified intelligence stack made up of three complementary layers:

| IQ Layer | Platform | Intelligence Type |
|---|---|---|
| **Fabric IQ** | Microsoft Fabric / OneLake | Structured data — ontologies, semantic models, graphs |
| **Foundry IQ** | Azure AI Foundry | Unstructured documents — policies, manuals, contracts |
| **Work IQ** | Microsoft 365 | Collaboration signals — emails, chats, meetings |

> These three layers are **not alternatives** — they solve different problems and are designed to work together. This document covers **Fabric IQ** and **Foundry IQ**.

---

## 3. Fabric IQ — The Structured Data Ontology

Fabric IQ is an intelligence workload inside **Microsoft Fabric** that introduces a formal **Ontology item** — a live, governed semantic layer sitting on top of your structured data in OneLake.

> **Official Definition (Microsoft Learn):**  
> *The Ontology (preview) item digitally represents the enterprise vocabulary and semantic layer that unifies meaning across domains and OneLake sources. It defines enterprise concepts as entity types (like Customer), properties (like a Customer's name and email), and relationships (like Customer places Order), while clarifying the constraints of these terms. Both humans and AI agents can use this language for cross-domain reasoning and decision-ready actions.*

---

### 3.1 Core Building Blocks

```
┌─────────────────────────────────────────────────────────────────┐
│                     FABRIC IQ ONTOLOGY                          │
│                                                                 │
│   Entity Types          Properties          Relationships       │
│   ─────────────         ──────────          ─────────────       │
│   Customer              .name               Customer            │
│   Order                 .email                 │ places         │
│   Product               .country               ▼               │
│   Shipment              .segment            Order               │
│   TempSensor            .currentTemp           │ contains       │
│                           (time series)        ▼               │
│                                            LineItem             │
│                         Data Bindings      │ references         │
│                         ─────────────      ▼                   │
│                         → Lakehouse        Product              │
│                         → Eventhouse                            │
│                         → Semantic Model                        │
└─────────────────────────────────────────────────────────────────┘
```

---

### 3.2 Entity Types

An **entity type** is the reusable logical model of a real-world concept. It standardises the name, description, identifiers, properties, and constraints so that every team means the same thing when they use a term like "Customer".

```yaml
# Entity Type definition (conceptual)
EntityType: Customer
  description: "A business entity that places orders with the company"
  primaryKey: customerId
  properties:
    - customerId:      String    [required, unique]
    - name:            String    [required]
    - email:           String    [unique, format: RFC5322]
    - country:         String
    - segment:         Enum      [Enterprise, SMB, Consumer]
    - lifetimeValue:   Decimal   [computed, min: 0]
  labels:
    - PII: [email, name]
    - Domain: Sales
```

> **Key insight:** By elevating a concept above any single table, entity types **eliminate conflicting definitions** across systems. A `Customer` is the same `Customer` whether its data comes from the CRM, the data warehouse, or the ERP.

---

### 3.3 Properties

Properties define the attributes of an entity type. Three kinds exist:

| Property Type | Description | Example |
|---|---|---|
| **Static** | Maps to a lakehouse table column — updated via batch ETL | `Customer.country → dim_customer.country_code` |
| **Time Series** | Maps to an eventhouse stream — updates continuously in real time | `Store.temperature → eventhouse: sensor_readings.store_temp` |
| **Computed** | Derived at query time via a formula defined in the ontology | `Order.totalValue = SUM(lineItem.price × lineItem.qty)` |

---

### 3.4 Relationship Types

Relationship types define directed semantic connections between entity types. They give the ontology its **graph structure** and enable **multi-hop traversal** — the ability to follow a chain of connections to answer complex questions.

```
Customer  ──[places]────────►  Order        (1 Customer : many Orders)
Order     ──[contains]──────►  LineItem     (1 Order : many LineItems)
LineItem  ──[references]────►  Product      (many LineItems : 1 Product)
Order     ──[dispatchedAs]──►  Shipment     (1 Order : 1 Shipment)
Shipment  ──[monitoredBy]───►  TempSensor   (1 Shipment : many Sensors)
```

**Example of a multi-hop query this enables:**

```gql
# "Find all Customers whose Shipment has a temperature reading above 8°C today"
Customer
  → [places] → Order
  → [dispatchedAs] → Shipment
  → [monitoredBy] → TempSensor
    WHERE TempSensor.reading > 8.0
    AND   TempSensor.timestamp = TODAY
```

---

### 3.5 Data Bindings

A **data binding** is the technical bridge that connects an abstract ontology definition to live data in OneLake. Without a binding, an entity type is just a schema definition. With a binding, it becomes a **live, queryable business object**.

```yaml
# Data Binding: Shipment entity → multiple OneLake sources
EntityType: Shipment

  # Static binding → lakehouse table
  StaticBinding:
    source:    lakehouse/gold_layer/fact_shipments
    key:       shipmentId = shipment_id
    columns:
      status:         → shipment_status
      origin:         → origin_warehouse_code
      destination:    → dest_location_code
      scheduledDate:  → eta_date

  # Time series binding → eventhouse stream (real-time)
  TimeSeriesBinding:
    source:   eventhouse/stream/cold_chain_sensors
    key:      shipmentId = sensor_shipment_ref
    property: currentTemp → sensor_temp_celsius

  # Data quality rules (enforced at concept layer)
  QualityRules:
    - currentTemp:    range [-30°C, 40°C]    # alert if breached
    - status:         not-null
    - scheduledDate:  not-past
```

**Binding supports three source types:**

| Source | Type | Use Case |
|---|---|---|
| Lakehouse tables | Static / batch | Core entity facts — customer master, product catalog |
| Eventhouse streams | Real-time / time series | Sensor readings, transaction events, IoT signals |
| Power BI semantic models | BI / analytical | KPI measures, hierarchy definitions, certified metrics |

---

### 3.6 How the Ontology Is Created

#### Method A — Auto-generate from a Power BI Semantic Model *(recommended starting point)*

Microsoft has 20M+ Power BI semantic models already encoding business logic. A single workspace action generates:

1. A new Ontology item in the Fabric workspace
2. Entity types matching every table in the semantic model
3. Static properties from each column with type mappings
4. Data bindings linking entity definitions to managed lakehouse tables
5. Relationship types following foreign key relationships

**Limitations of auto-generation:**

- Only managed lakehouse tables in the same OneLake directory (no external tables)
- Delta tables with column mapping enabled are not supported
- Time series bindings (eventhouse) must be added manually after generation
- XMLA endpoint limitations of the source semantic model apply

#### Method B — Author in Ontology Manager *(manual / greenfield)*

For organisations without existing Power BI models, **Ontology Manager** provides a low-code visual tool to:

- Define entity types, properties, and relationship types
- Configure data bindings to OneLake sources
- Set constraints, quality rules, and labels
- Version, review, and publish ontology changes

---

### 3.7 Full Fabric IQ Architecture

Every item in the Fabric IQ workload depends on the Ontology as its foundation:

```
                    ┌─────────────────┐
                    │   ONTOLOGY      │  ◄── Foundation layer
                    │  (entity types, │
                    │   relationships,│
                    │   data bindings)│
                    └────────┬────────┘
        ┌───────────┬────────┼────────┬───────────┐
        ▼           ▼        ▼        ▼           ▼
  ┌──────────┐ ┌────────┐ ┌──────┐ ┌──────┐ ┌────────┐
  │ Semantic │ │ Graph  │ │ Data │ │ Ops  │ │  Plan  │
  │  Model   │ │Engine  │ │Agent │ │Agent │ │        │
  └──────────┘ └────────┘ └──────┘ └──────┘ └────────┘
  KPI defs     Traversal  NL query  Monitor   Planning
  + BI reports  engine    + answer  + act     + writeback
```

| Item | Role | How It Uses the Ontology |
|---|---|---|
| **Ontology** | Central semantic layer | Defines all entity types, properties, relationships, bindings |
| **Semantic Model** | Trusted BI definitions | Source for auto-generation; keeps KPIs consistent |
| **Graph** | Relationship traversal | Ontology declares what connects; Graph computes traversals |
| **Data Agent** | NL-to-ontology query | Resolves natural language into precise ontology queries |
| **Operations Agent** | Real-time monitoring + actions | Fires governed actions when ontology property rules are breached |
| **Plan** | Planning + forecasting | Uses ontology entities as planning objects with writeback |
| **Fabric Activator** | Event-driven trigger | Ontology rules define conditions; Activator fires alerts |

---

### 3.8 End-to-End Technical Flow

```
Question: "Which customers are at risk of a cold chain breach today?"

─────────────────────────────────────────────────────────────────

STEP 1 — DATA IN ONELAKE
  ├── Lakehouse:   gold_layer.dim_customer     (batch, nightly)
  ├── Lakehouse:   gold_layer.fact_shipments   (batch, hourly)
  └── Eventhouse:  cold_chain_sensors          (streaming, real-time)

─────────────────────────────────────────────────────────────────

STEP 2 — ONTOLOGY RESOLVES LIVE INSTANCES
  Customer(id=C001, name="Contoso", segment="Enterprise")
  Shipment(id=S123, status="In Transit", currentTemp=9.2°C)
                                         ▲ from eventhouse stream
  Relationship: C001 ──[places]──► O456
  Relationship: O456 ──[dispatched]──► S123

─────────────────────────────────────────────────────────────────

STEP 3 — RULE EVALUATION
  Constraint: Shipment.currentTemp <= 8.0°C
  Shipment S123: currentTemp = 9.2°C  →  ❌ BREACH CONDITION MET

─────────────────────────────────────────────────────────────────

STEP 4 — DATA AGENT RESOLUTION
  NL input:    "customers at risk of cold chain breach today"
  Resolved to: Customer entity
               ↓ [places] ↓ Order
               ↓ [dispatchedAs] ↓ Shipment[currentTemp > 8.0]
  Graph traversal executes across ontology-declared relationships

─────────────────────────────────────────────────────────────────

STEP 5 — GROUNDED ANSWER
  "Customer Contoso (Enterprise) has 1 active shipment (S123),
   currently at 9.2°C — exceeding the 8.0°C cold chain limit.
   Shipment origin: Frankfurt. ETA: 2026-05-16."

  Sources cited:
    [Ontology:Customer] [Ontology:Shipment]
    [Stream:cold_chain_sensors] [Rule:TempConstraint]
```

---

### 3.9 Security and Governance

| Control | Mechanism |
|---|---|
| Access control | Microsoft Entra ID + Fabric workspace roles |
| Column-level sensitivity | Microsoft Purview labels flow through data bindings |
| Ontology versioning | Changes are versioned, reviewable, and rollback-capable |
| Schema evolution | Data quality rules enforced at concept layer (not just table level) |
| Cross-tenant sharing | OneLake external data sharing for governed cross-boundary access |
| Audit | Fabric activity log captures all ontology reads and modifications |

---

## 4. Foundry IQ — The Unstructured Knowledge Layer

> ⚠️ **Critical distinction:** Foundry IQ does **not** contain an entity/relationship ontology like Fabric IQ. It is a **managed knowledge retrieval system** — built on Azure AI Search — that grounds AI agents in unstructured enterprise content: documents, policies, contracts, manuals, wikis, and web pages.

---

### 4.1 The Problem It Solves

Before Foundry IQ, every team building an AI agent over company documents had to rebuild the entire retrieval pipeline from scratch:

```
Old approach (per-agent, per-project):
  Agent A:  [custom connectors] → [custom chunking] → [custom embeddings] → [custom routing]
  Agent B:  [custom connectors] → [custom chunking] → [custom embeddings] → [custom routing]
  Agent C:  [custom connectors] → [custom chunking] → [custom embeddings] → [custom routing]
             ▲ duplicated        ▲ duplicated          ▲ duplicated          ▲ duplicated
             ▲ inconsistent      ▲ inconsistent         ▲ inconsistent        ▲ inconsistent
```

**Foundry IQ approach:**

```
  Define knowledge base once:
    KnowledgeBase ──────────────────────────────────────────────►
     ├── SharePoint (policies)                              Agent A
     ├── Blob Storage (manuals)                             Agent B
     ├── OneLake (Fabric IQ structured data)                Agent C
     └── Web (public standards)                             Agent N
```

---

### 4.2 Knowledge Base Architecture

```yaml
# Knowledge Base (conceptual structure)
KnowledgeBase: "procurement-policies-kb"

  KnowledgeSources:

    # Source 1 — Indexed (auto-managed pipeline)
    - type:       SharePoint
      site:       "https://contoso.sharepoint.com/sites/procurement"
      scope:      "/Policies, /Contracts, /Standards"
      indexer:    auto                          # chunking + vectorisation
      schedule:   incremental-refresh-24h
      enrichment: AzureContentUnderstanding     # layout-aware: tables, headers
      permissions: Entra-ID-document-level      # per-user document ACL enforcement

    # Source 2 — Indexed
    - type:       AzureBlobStorage
      container:  "vendor-manuals"
      indexer:    auto
      enrichment: AzureContentUnderstanding

    # Source 3 — Remote (queried live, not re-indexed)
    - type:       OneLake                       # Fabric IQ structured data passthrough
      endpoint:   "https://onelake.dfs.fabric.microsoft.com/..."

    # Source 4 — Remote
    - type:       Web
      scope:      public                        # retrieved at query time

  RetrievalBehaviour:
    mode:              hybrid                   # keyword + vector search
    reasoning_effort:  medium                   # enables iterative search
    return_citations:  true
    permissions:       Entra-ID
```

---

### 4.3 Agentic Retrieval Engine

The agentic retrieval engine treats retrieval as a **multi-step reasoning task**, not a single keyword lookup.

```
Query: "What does our Q4 procurement policy say about single-source suppliers from Germany?"

┌─────────────────────────────────────────────────────────────────┐
│  STAGE 1 — PLAN                                                 │
│  LLM decomposes the query into sub-queries:                     │
│    sub-query 1: "Q4 procurement policy single-source rule"      │
│    sub-query 2: "Germany supplier exemptions procurement"       │
│    sub-query 3: "single-source approval threshold spend limit"  │
│  Selects sources: SharePoint/Policies, Blob/Contracts           │
└───────────────────────┬─────────────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────────────┐
│  STAGE 2 — SEARCH (parallel)                                    │
│  Sub-query 1 → hybrid search on SharePoint/ProcurementPolicy    │
│  Sub-query 2 → hybrid search on Blob/SupplierCodeOfConduct      │
│  Sub-query 3 → hybrid search on both sources                    │
└───────────────────────┬─────────────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────────────┐
│  STAGE 3 — RANK                                                 │
│  Semantic reranking across all results from all sources         │
│  Deduplication + low-quality result removal                     │
└───────────────────────┬─────────────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────────────┐
│  STAGE 4 — REFLECT  (triggered when results are insufficient)   │
│  Engine issues a follow-up query:                               │
│    "single source Germany spend approval threshold 2026"        │
│  Iterates until confidence threshold is met                     │
└───────────────────────┬─────────────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────────────┐
│  STAGE 5 — SYNTHESISE                                           │
│  Returns a grounded answer with per-claim citations:            │
│    "German suppliers are exempt from dual-sourcing [¹]"         │
│    "Single-source approval required above €1.5M [²]"           │
│    [¹] ProcurementPolicy_Q4_2026.pdf, page 4                   │
│    [²] SupplierCodeOfConduct_v3.docx, section 7.2              │
└─────────────────────────────────────────────────────────────────┘
```

---

### 4.4 Supported Knowledge Sources

| Source | Indexing | Permission Model | Notes |
|---|---|---|---|
| Azure Blob Storage | Auto (indexed) | Azure RBAC | PDFs, Word, text, HTML |
| SharePoint Online | Auto (indexed) | Entra ID document-level | Sites and document libraries |
| OneLake (Fabric IQ) | Remote (live query) | Fabric workspace roles | Structured data passthrough |
| Azure AI Search index | Federated (remote) | Search API key / Entra | Existing search indexes |
| Public Web | Remote (live query) | None | Retrieved at query time |
| MCP Servers | Remote (live query) | Per-MCP config | Private preview, May 2026 |

---

### 4.5 Integration Code Example

#### Python — Agent Framework

```python
import asyncio
from agent_framework import ChatAgent
from agent_framework.azure import AzureAIAgentClient, AzureAISearchContextProvider
from azure.identity.aio import DefaultAzureCredential

async def main():
    credential = DefaultAzureCredential()  # Managed Identity — no keys stored

    async with (
        # Connect to Foundry IQ Knowledge Base
        AzureAISearchContextProvider(
            endpoint             = "https://<search-service>.search.windows.net",
            knowledge_base_name  = "procurement-policies-kb",
            credential           = credential,
            mode                 = "agentic",   # enables multi-hop retrieval
        ) as knowledge_base,

        # Connect to Azure OpenAI model
        AzureAIAgentClient(
            project_endpoint      = "https://<project>.services.ai.azure.com/api/projects/<id>",
            model_deployment_name = "gpt-4o",
            async_credential      = credential,
        ) as model_client,

        # Create the grounded agent
        ChatAgent(
            chat_client       = model_client,
            context_providers = [knowledge_base]
        ) as agent,
    ):
        answer = await agent.run(
            "What does our Q4 policy say about single-source suppliers from Germany?"
        )
        print(answer)  # Grounded answer with citations

asyncio.run(main())
```

#### REST API — Azure AI Search (agentic retrieval)

```http
POST https://<search-service>.search.windows.net
     /knowledgebases/<kb-name>/retrieve?api-version=2026-04-01

Content-Type: application/json
Authorization: Bearer <Entra-ID-token>

{
  "messages": [
    {
      "role": "user",
      "content": "What are the single-source supplier rules for Germany in Q4?"
    }
  ],
  "retrieval_options": {
    "reasoning_effort": "medium",
    "max_docs_for_reranker": 50
  }
}
```

---

### 4.6 Security and Governance

| Control | Mechanism |
|---|---|
| Document-level access | Entra ID permissions respected per source at query time |
| No cross-user data leak | Retrieval scoped to calling user's identity — never cached cross-user |
| Sensitivity labels | Microsoft Purview labels flow through retrieval results |
| Audit logging | All queries logged in Azure Monitor |
| Compliance | Built on Azure AI Search — ISO 27001, SOC 2, GDPR certified |
| API versioning | Stable API: `2026-04-01` · Preview API: `2025-11-01-preview` |

---

## 5. Fabric IQ vs Foundry IQ — Direct Comparison

| Dimension | Fabric IQ Ontology | Foundry IQ Knowledge Base |
|---|---|---|
| **Purpose** | Formal shared understanding of structured business data | Permission-aware answers from unstructured enterprise documents |
| **Data type** | Structured (lakehouse) + streaming (eventhouse) + BI models | Unstructured: PDFs, Word, SharePoint, wikis, contracts, web |
| **Core concept** | Entity types · Properties · Relationship types · Data bindings | Knowledge Base · Knowledge Sources · Agentic Retrieval Engine |
| **Semantic layer** | Formal entity/relationship ontology — explicitly defined by humans | Implicit semantic grounding — meaning extracted by AI from text |
| **AI grounding method** | Agent queries ontology → typed business objects with provenance | Agent queries knowledge base → document excerpts with citations |
| **Built on** | Microsoft Fabric / OneLake / Power BI semantic models | Azure AI Search (agentic retrieval API) |
| **Where it lives** | Fabric workspace as an `Ontology` item | Microsoft Foundry as a `KnowledgeBase` resource |
| **Auto-creation** | ✅ Generate from Power BI semantic model in one click | ✅ Auto-indexes, chunks, and vectorises content automatically |
| **Real-time data** | ✅ Eventhouse streams bind to entity properties (time series) | ❌ Indexed sources refresh on schedule; remote sources are live |
| **Write-back / actions** | ✅ Operations Agent + Fabric Activator trigger governed actions | ❌ Retrieval only — no action execution |
| **Multi-hop reasoning** | ✅ Graph engine traverses ontology-declared relationship types | ✅ Agentic retrieval Plan-Search-Reflect loop |
| **Permissions** | Fabric workspace roles + Entra ID + Purview column labels | Entra ID document-level per knowledge source |
| **Status (May 2026)** | Public Preview — evolving rapidly | Public Preview — GA planned Q2 2026 |

---

## 6. How They Work Together

OneLake (Fabric IQ) is a **native knowledge source** in Foundry IQ, meaning a single agent can simultaneously:

- Query the **Fabric IQ Ontology** for structured entity facts
- Query the **Foundry IQ Knowledge Base** for relevant policy/document content
- Combine both into one grounded, cited answer

### Combined Agent Example

```
Question: "Is Supplier XYZ compliant with our Q4 procurement policy?"

══════════════════════════════════════════════════════════════════
  FABRIC IQ — Structured Ontology Query
══════════════════════════════════════════════════════════════════

  Supplier entity (from ERP via lakehouse binding):
    supplierId:    XYZ-001
    country:       Germany
    category:      Raw Materials
    totalSpend_Q4: €2.4M         ← from fact_procurement (lakehouse)
    riskScore:     3.2           ← computed property (risk model)

  Relationship traversal:
    Supplier XYZ ──[linkedTo]──► Contract #C-882
    Supplier XYZ ──[hasOrders]──► PurchaseOrder[] (14 orders in Q4)

══════════════════════════════════════════════════════════════════
  FOUNDRY IQ — Policy Document Retrieval
══════════════════════════════════════════════════════════════════

  Knowledge Base: "procurement-policies-kb"
  Sources queried:
    SharePoint/ProcurementPolicy_Q4_2026.pdf
    AzureBlob/SupplierCodeOfConduct_v3.docx

  Retrieved:
    [¹] "All suppliers in category Raw Materials with spend > €2M must
        submit a Tier-2 environmental audit by March 31 each year."
    [²] "German suppliers are exempt from dual-sourcing requirements.
        Single-source approval required for spends above €1.5M."

══════════════════════════════════════════════════════════════════
  COMBINED AGENT ANSWER
══════════════════════════════════════════════════════════════════

  "Supplier XYZ (Germany, Raw Materials) spent €2.4M in Q4 — exceeding
   the €2M Tier-2 audit threshold [¹]. No audit record found.

   ⚠️  ACTION REQUIRED: Request Tier-2 environmental audit submission.
   ℹ️  Note: German single-source exemption applies [²] —
       no dual-sourcing action needed for this supplier."

  Sources: [Ontology:Supplier] [Ontology:PurchaseOrder]
           [¹] ProcurementPolicy_Q4_2026.pdf, page 6
           [²] SupplierCodeOfConduct_v3.docx, section 7.2
```

### The Three IQ Layers — Full Picture

```
                ENTERPRISE AI AGENT
                       │
      ┌────────────────┼────────────────┐
      ▼                ▼                ▼
┌──────────┐     ┌──────────┐     ┌──────────┐
│ WORK IQ  │     │ FABRIC   │     │ FOUNDRY  │
│          │     │   IQ     │     │   IQ     │
│ M365     │     │          │     │          │
│ Emails   │     │ Ontology │     │Knowledge │
│ Chats    │     │ Entities │     │   Base   │
│ Meetings │     │ KPIs     │     │ Policies │
│ Files    │     │ Streams  │     │ Manuals  │
└──────────┘     └──────────┘     └──────────┘
"What have      "What are the    "What do our
 people said     facts about      documents
 about this?"    this entity?"    say?"
```

---

## 7. Decision Guide — Which Layer for Which Problem?

| Problem / Use Case | Use This Layer | Why |
|---|---|---|
| Copilot gives different "revenue" figures across teams | Fabric IQ Ontology | Revenue is a structured metric — define it once as a measure on the Order entity |
| Agent can't find our HR leave policy | Foundry IQ Knowledge Base | Leave policy is an unstructured PDF — index it in a knowledge base |
| Real-time alert when cold chain temperature exceeds a threshold | Fabric IQ Ontology | Time series property on Shipment entity; Fabric Activator fires on breach |
| Agent must answer questions from 50 SharePoint manuals | Foundry IQ Knowledge Base | Unstructured content — auto-indexed with Azure Content Understanding |
| Different departments use different names for the same customer | Fabric IQ Ontology | Semantic drift is resolved by defining Customer once as an entity type |
| Agent must cite a contract clause before recommending action | Foundry IQ Knowledge Base | Contract is a PDF — retrieval surfaces the relevant clause with citation |
| Power BI reports show inconsistent KPIs | Fabric IQ Ontology | Generate from the canonical semantic model; all reports align to the same definitions |
| Multi-hop: "Which customers are affected by delayed shipments with temp breaches?" | Fabric IQ Ontology + Graph | Customer → Order → Shipment[temp > threshold] — Graph traverses ontology relationships |
| Agent needs structured supplier facts AND policy documents | **Both layers** | Fabric IQ provides entity facts; Foundry IQ provides policy text |
| Custom developer-built agent outside the Fabric workspace | Foundry IQ (primary) | Agent Framework + Azure AI Search APIs callable from any Azure-hosted app |

---

## References

| Resource | URL |
|---|---|
| Microsoft Learn — What Is Ontology (Preview) | https://learn.microsoft.com/en-us/fabric/iq/ontology/overview |
| Microsoft Learn — What Is Fabric IQ (Preview) | https://learn.microsoft.com/en-us/fabric/iq/overview |
| Microsoft Learn — What Is Foundry IQ | https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/what-is-foundry-iq |
| Microsoft Learn — Agentic Retrieval Overview | https://learn.microsoft.com/en-us/azure/search/agentic-retrieval-overview |
| Microsoft Learn — Generate Ontology from Semantic Model | https://learn.microsoft.com/en-us/fabric/iq/ontology/concepts-generate |
| Microsoft Fabric Blog — Introducing Fabric IQ | https://blog.fabric.microsoft.com/en-us/blog/from-data-platform-to-intelligence-platform-introducing-microsoft-fabric-iq |
| Azure AI Tech Community — Foundry IQ | https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/foundry-iq-unlocking-ubiquitous-knowledge-for-agents/4470812 |

---

*Last updated: May 2026 · Fabric IQ Ontology — Public Preview · Foundry IQ — Public Preview*
