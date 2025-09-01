# Text-to-SQL with Vanna, NVIDIA NIM, Milvus, and SAP HANA

This guide outlines building a robust Text-to-SQL system leveraging Vanna, NVIDIA NIM, and Milvus, designed specifically for SAP HANA as the target SQL database.

## 1. Overview
Your system will:
- Intelligently interpret natural language queries  
- Retrieve schema context and documentation via Milvus and NVIDIA embeddings  
- Generate SQL through an LLM served by NVIDIA NIM  
- Execute queries against SAP HANA and return results  

The architecture separates retrieval and generation for modularity and performance.

## 2. Architecture
- User query submitted in plain language  
- Milvus (vector DB) retrieves relevant context using NVIDIA embeddings  
- NIM-hosted LLM (via OpenAI-compatible interface) generates the SQL  
- SQL is executed on SAP HANA, and results are returned  

---

# Deployment Guide

## 1. Setup SAP HANA Environment

- Pull and run the SAP HANA Express Docker container:

```bash
sudo docker run -d --name hanaexpress --restart unless-stopped --ulimit nofile=1048576:1048576 --sysctl kernel.shmmax=1073741824 --sysctl kernel.shmall=8388608 -p 39013:39013 -p 39015:39015 -p 39017:39017 -v /data/hana:/hana/mount --security-opt seccomp=unconfined saplabs/hanaexpress:latest --passwords-url file:///hana/mount/password.json
```

- Verify that SAP HANA is running and accessible on the mapped ports (usually 39015 for SQL).

---

## 2. Install Python Environment & Dependencies

- Use Python 3.10+.  
- Install essential Python packages:

```bash
pip install vanna milvus hdbcli pandas
```

- **vanna** for orchestration and RAG framework.  
- **milvus** as the vector store.  
- **hdbcli** for SAP HANA connectivity.  
- **pandas** for data handling.  

---

## 3. Clone NVIDIA Generative AI Examples with Vanna

```bash
git clone https://github.com/NVIDIA/GenerativeAIExamples.git
cd GenerativeAIExamples/community/Vanna_with_NVIDIA_AI_Endpoints
```

- Open the notebook: **Vanna_with_NVIDIA.ipynb**.  
- Specify your NVIDIA API key in the notebook:

```python
nvidia_api_key = 'nvapi-...'
```

- To get the NVIDIA API key:
  - Register at [build.nvidia.com](https://build.nvidia.com/)  
  - Log in  
  - Navigate to API catalog  
  - Generate a key  

---

## 4. Extract and Ingest SAP HANA Schema & Docs into Milvus

- Extract SAP HANA DDL schema using SQL:

```sql
SELECT TABLE_NAME, COLUMN_NAME, DATA_TYPE_NAME
FROM SYS.TABLE_COLUMNS
WHERE SCHEMA_NAME = 'YOUR_SCHEMA';
```

- Collect any business definitions, metric descriptions, and example queries.  
- Chunk and embed these texts with NVIDIA embeddings.  
- Upload embedded vectors and metadata into Milvus as the context store.  

---

## 5. Configure Vanna Pipeline

- Create a custom connection function to execute SQL on SAP HANA:

```python
from hdbcli import dbapi
import pandas as pd

def hana_query(sql):
    conn = dbapi.connect(
        address="localhost",
        port=39015,
        user="SYSTEM",
        password="YOUR_PASSWORD"
    )
    df = pd.read_sql(sql, conn)
    conn.close()
    return df
```

- Initialize Vanna with Milvus as vector store and NVIDIA NIM (via OpenAI-compatible interface) as LLM client.  
- Set SAP HANA SQL dialect and apply guardrails for:  
  - Read-only queries  
  - Default row limits  
  - Masking sensitive columns  

---

## 6. Train Vanna RAG Model

- Train on schema DDL, documentation, and relevant few-shot examples:

```python
vanna.train(ddl_text)
vanna.train(documentation_text)
vanna.train(few_shot_example)
```

- Tune chunk sizes and retrieval settings for precision.  

---

## 7. Runtime Query Flow

1. User submits natural language queries.  
2. Vanna retrieves schema context from Milvus.  
3. The LLM hosted on NVIDIA NIM generates SQL queries.  
4. SQL executes in SAP HANA via the custom connector.  
5. Query results are returned as Pandas DataFrame or visualized.  

---

## 8. Production Best Practices

- Enforce **SQL safety**: read-only mode, dialect validation, limits.  
- Use **secure connections** and least-privilege SAP HANA accounts.  
- Monitor **latency, retrieval accuracy, error rates, and logs**.  
- Cache embeddings and enable **streaming from NIM** for performance.  
- Use **autoscaling** for Milvus and NIM microservices.  
- Implement **schema change detection** for re-ingestion.  
- Manage **secrets** (API keys, passwords) securely.  
- Conduct **canary testing** for embedding and model updates.  

---

## Summary Table

| **Step**                 | **Description**                                      | **Tools/Tech**                        |
|---------------------------|------------------------------------------------------|---------------------------------------|
| SAP HANA Setup            | Run Express edition in Docker                        | Docker, SAP HANA                      |
| Python Environment Setup  | Install packages                                     | Python, pip                           |
| Repo Setup & API Key      | Clone repo, set NVIDIA API Key                       | Git, NVIDIA build                     |
| Schema Extraction         | Extract DDL and documentation                        | SAP HANA SQL                          |
| Milvus Context Ingestion  | Embed and upload schema/docs to Milvus               | NVIDIA Embeddings, Milvus             |
| Vanna Configuration       | Initialize Vanna with Milvus and NIM LLM             | Vanna, NVIDIA NIM                     |
| RAG Model Training        | Train on DDL, docs, examples                         | Vanna API                             |
| Querying & Execution      | User queries → retrieval → SQL gen → SAP HANA exec   | Vanna, SAP HANA                       |
| Production QA & Security  | Guardrails, monitoring, secret mgmt                  | All components                        |

---

This guide enables deploying a **modular, robust Text-to-SQL pipeline** that takes natural language queries, retrieves context from SAP HANA schema and docs with Milvus, generates dialect-safe SQL via NVIDIA-powered LLMs, and executes queries on SAP HANA securely with performant results.
