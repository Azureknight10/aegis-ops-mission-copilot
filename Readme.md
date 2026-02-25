Here is a first-pass `README.md` aligned with your phase plan and naming.

***

# AEGIS‑OPS Mission Copilot

AEGIS‑OPS Mission Copilot is a defense‑flavored operations assistant that helps Field Engineers and Cloud SREs triage incidents, execute SOPs, and reason about complex systems using Azure OpenAI, Azure AI Search, and a lightweight mission console UI. [github](https://github.com/Azure-Samples/azure-search-openai-demo)

## Goals

- Provide a focused “mission console” for on‑call scenarios (field and cloud).  
- Use Retrieval‑Augmented Generation (RAG) over SOPs and incident reports to generate grounded answers with citations. [github](https://github.com/Azure-Samples/azure-ai-vercel-rag-starter)
- Demonstrate an end‑to‑end Azure reference implementation (data → API → AI → web → CI/CD).

***

## Personas

### FieldEngineer

Top questions:

- “Given these symptoms in the field, which runbook should I follow and what are the exact steps?”  
- “Is there a known incident that looks like this one and how was it resolved?”  
- “What is safe to try in this environment (lab vs staging vs production)?”

### CloudSRE

Top questions:

- “What recent incidents are related to this subsystem or service?”  
- “How have we fixed this class of failure before and what was the root cause?”  
- “What are the next best actions that minimize blast radius for this phase of the mission?”

***

## High‑level Architecture

Planned Azure services:

- **Azure OpenAI Service** – chat completions and embeddings. [github](https://github.com/Azure-Samples/app-service-rag-openai-ai-search-nodejs)
- **Azure AI Search** – indexing and retrieval over SOPs and incidents. [github](https://github.com/Azure-Samples/azure-search-openai-demo)
- **Azure Storage Account** – source of truth for SOP/incident JSON and any future artifacts. [github](https://github.com/Azure-Samples/app-service-rag-openai-ai-search-python)
- **Azure App Service** – hosts the API (and optionally the web app). [github](https://github.com/Azure-Samples/app-service-rag-openai-ai-search-nodejs)
- **Azure Application Insights** – telemetry for API, chat usage, and basic performance. [github](https://github.com/Azure-Samples/app-service-rag-openai-ai-search-python)

### Text Architecture Diagram (ASCII)

```text
+------------------------------+
|   FieldEngineer / CloudSRE   |
|      (Web Mission UI)        |
+--------------+---------------+
               |
               v
       +-------+--------+
       |   AEGIS-OPS    |
       |   API (Node)   |
       +---+-------+----+
           |       |
           |       +---------------------+
           |                             |
           v                             v
+--------------------+         +----------------------+
|  Azure AI Search   |<------->|   Azure OpenAI       |
|  (SOPs/Incidents)  |         | (chat + embeddings)  |
+---------+----------+         +----------------------+
          |
          v
+------------------------+
|  Storage / Git-backed  |
|  JSON SOPs & Incidents |
+------------------------+

Observability: App Insights wired to API and key operations.
CI/CD: GitHub Actions → App Service (API + Web).
```

***

## Repository Layout

Planned structure:

```text
/
├─ tracker.html              # Lightweight build tracker
├─ README.md
├─ /infra                    # Bicep/Terraform for Azure resources
├─ /api                      # Node.js / TypeScript backend
├─ /web                      # React/Vite mission console UI
└─ /data
   ├─ /schema                # JSON schema definitions
   │  ├─ sop.schema.json
   │  └─ incident.schema.json
   ├─ /sops                  # Synthetic SOP documents
   └─ /incidents             # Synthetic incident/AAR documents
```

***

## Data Model

### SOP Schema (`/data/schema/sop.schema.json`)

Core fields:

- `id`: Unique SOP identifier (string).  
- `title`: Human‑readable SOP title.  
- `system_id`: Logical system identifier (e.g., AEGIS‑01).  
- `subsystem`: Sub‑component or capability (e.g., telemetry, comms, targeting).  
- `severity`: Severity level the SOP is designed for (e.g., SEV1–SEV3).  
- `environment`: Target environment (lab, staging, prod, field).  
- `phase`: Mission/incident lifecycle phase (detect, triage, mitigate, recover, postmortem).  
- `steps[]`: Ordered list of steps, each with step text and optional metadata.  
- `related_incident_ids[]`: List of incident IDs this SOP has been applied to.

### Incident Schema (`/data/schema/incident.schema.json`)

Core fields:

- `id`: Unique incident identifier.  
- `system_id`: System involved in the incident.  
- `symptoms`: Text description of observed symptoms.  
- `root_cause`: Text description of confirmed or suspected root cause.  
- `phase`: Incident phase at the time of record (e.g., detect, triage, mitigate, recover, postmortem).  
- `environment`: Environment where the incident occurred (lab, staging, prod, field).

See `DATA_NOTES.md` for notes on synthetic content.

***

## API (Backend)

The backend is a Node.js / TypeScript API under `/api` that exposes health, catalog, and chat endpoints.

### Planned Endpoints

- `GET /health` – health probe for App Service and CI.  
- `GET /sops` – returns SOPs loaded from `/data/sops`.  
- `GET /incidents` – returns incidents loaded from `/data/incidents`.  
- `POST /chat` – RAG‑backed mission chat endpoint that:  
  - Enriches the user prompt with persona (FieldEngineer / CloudSRE).  
  - Queries Azure AI Search over SOPs/incidents. [github](https://github.com/Azure-Samples/azure-ai-vercel-rag-starter)
  - Calls Azure OpenAI to generate grounded answers with citations to SOPs/incidents. [github](https://github.com/Azure-Samples/azure-search-openai-demo)

Basic logging records request path, status code, and chat usage for observability.

***

## Web UI

The web mission console is a React/Vite app under `/web`:

- Persona switcher: toggle between FieldEngineer and CloudSRE modes.  
- Chat panel: sends queries to `POST /chat` and renders streaming responses. [github](https://github.com/AzureCosmosDB/cosmosdb-nosql-copilot)
- Citations panel: shows which SOPs/incidents were used for the answer. [github](https://github.com/Azure-Samples/app-service-rag-openai-ai-search-nodejs)
- Runbook view: renders SOP steps as a checklist for execution.

***

## Azure Resources & Deployment

### Planned Azure Resources

- Azure App Service (API and optionally web). [github](https://github.com/Azure-Samples/app-service-rag-openai-ai-search-python)
- Azure AI Search service and index for SOPs/incidents. [github](https://github.com/Azure-Samples/azure-ai-vercel-rag-starter)
- Azure OpenAI resource with:  
  - Chat model deployment.  
  - Embedding model deployment. [github](https://github.com/Azure-Samples/azure-search-openai-demo)
- Storage account for documents and future artifacts. [github](https://github.com/Azure-Samples/app-service-rag-openai-ai-search-nodejs)
- Application Insights for telemetry. [github](https://github.com/Azure-Samples/app-service-rag-openai-ai-search-python)

Infrastructure templates (Bicep or Terraform) will live under `/infra`, defining names, locations, and core configuration.

***

## CI/CD

GitHub Actions will be used to:

- Build and test the API on every push.  
- Deploy the API to Azure App Service (or Container Apps). [github](https://github.com/Azure-Samples/azure-search-openai-demo-csharp)
- Build and deploy the web app (static hosting or App Service). [github](https://github.com/Azure-Samples/app-service-rag-openai-ai-search-nodejs)

Secrets (Azure credentials, API keys, connection strings) are injected via GitHub Actions secrets and App Service configuration, not committed to source.

***

## How This Maps to Azure Patterns

AEGIS‑OPS Mission Copilot implements a standard Retrieval‑Augmented Generation (RAG) architecture on Azure:

- Documents (SOPs and incidents) live as JSON and are indexed into Azure AI Search. [github](https://github.com/Azure-Samples/azure-ai-vercel-rag-starter)
- Queries from the mission console hit an API that orchestrates search + OpenAI. [github](https://github.com/Azure-Samples/azure-search-openai-demo-csharp)
- Azure OpenAI generates answers using retrieved context and returns citations. [github](https://github.com/Azure-Samples/azure-search-openai-demo)
- The whole stack is hosted on Azure App Service with observability via Application Insights. [github](https://github.com/Azure-Samples/app-service-rag-openai-ai-search-python)

This mirrors official Azure RAG samples for OpenAI + AI Search, adapted to an operations/mission‑control scenario. [github](https://github.com/AzureCosmosDB/cosmosdb-nosql-copilot)

***

## Business Outcomes

Examples of business outcomes this copilot can enable:

- Faster incident triage by surfacing relevant SOPs and similar past incidents in a single mission console. [github](https://github.com/AzureCosmosDB/cosmosdb-nosql-copilot)
- Reduced cognitive load for on‑call Field Engineers and SREs by converting unstructured runbooks into guided checklists. [github](https://github.com/AzureCosmosDB/cosmosdb-nosql-copilot)
- Better post‑incident learning by tying SOPs directly to incidents/AARs and making them easily discoverable in context. [github](https://github.com/AzureCosmosDB/cosmosdb-nosql-copilot)

***

## Synthetic Data Notice

All SOPs, incidents, and mission narratives in this repository are fully synthetic and do not represent real systems, customers, or operations. See `DATA_NOTES.md` for details.