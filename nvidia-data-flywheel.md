<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Convert this to markdown script

Here is the content from "NVIDIA-Data-Flywheel.docx" converted into a well-structured Markdown script.[^1]

***

# NVIDIA Data Flywheel

## What the Data Flywheel Is

A **Data Flywheel** is an automated workflow that leverages production traffic (prompt/response logs, user feedback) to create evaluation and fine-tuning datasets. It runs experiments on candidate models (with different prompts, few-shot or fine-tuned variants), evaluates them, and surfaces promising, smaller or cheaper models for human review and potential promotion to production. The goal is to reduce latency and inference cost while maintaining accuracy targets.

References:

- [GitHub](https://github.com/NVIDIA-AI-Blueprints/data-flywheel)
- [NVIDIA Developer Blog](https://developer.nvidia.com/blog/build-efficient-ai-agents-through-model-distillation-with-nvidias-data-flywheel-blueprint/)

***

## High-level Flow

1. Production app emits logs: prompt, response, metadata (workload_id, client_id)
2. Logs centralized (e.g., Elasticsearch)
3. Flywheel pulls logs → creates stratified eval and fine-tune datasets → runs experiments across candidate NIMs and fine-tune strategies (LoRA, few-shot/ICL, etc.)
4. Results evaluated (LLM-as-judge similarity, other metrics)
5. Flywheel surfaces promising models for human inspection and possible production promotion

***

## Core Components

- **Log Store:** Central repository for prompt/response documents (reference uses Elasticsearch)
- **NeMo Datastore:** Curated datasets from logs
- **NeMo Customizer:** Fine-tuning workflows (LoRA, etc.)
- **NeMo Evaluator:** Automated evaluations (LLM-as-judge metrics)
- **NeMo Deployment Manager / NIMs:** Runs NIMs for evaluation and benchmarking
- **Orchestrator / Flywheel API:** Schedules dataset creation, experiments, tracks results
- **Auxiliary Infrastructure:** MongoDB (metadata), Redis (task queue/caching), Kubernetes or Docker for orchestration

***

## Log Schema (Minimal Fields)

- `timestamp` (int, epoch seconds): When request issued
- `workload_id` (string): Stable identifier for logical task/agent node
- `client_id` (string): Which application/customer/environment produced traffic
- `request` (dict): Model request payload (e.g., ChatCompletion payload)
- `response` (dict): Model response payload

**Note:** `workload_id` is crucial for splitting traffic into tasks for independent evaluation.

***

## Data Preparation \& Dataset Creation

- **Slice by workload:** Group traffic by `workload_id`; each becomes its own dataset target.
- **De-dup \& class-aware stratification:** Deduplicate examples and ensure balanced representation (important for multi-intent workloads).
- **Create eval sets:** Automated creation of eval sets—base prompts, ICL prompts, fine-tune test splits. Three experiment families per NIM: base, icl (few-shot), customized (LoRA/fine-tune).

***

## Experiment Types \& Comparison

- **base:** Production prompts replayed against candidate NIMs (no few-shot)
- **icl:** Few-shot/in-context learning prompts from traffic examples
- **customized:** LoRA or other fine-tune adapters evaluated on base eval set

**Evaluation Metric:** LLM-as-judge similarity score () to compare outputs to baseline. Models are ranked by similarity, latency, and cost; smaller/cost-efficient models are surfaced if they meet accuracy thresholds.[^1]

***

## Typical Deployment Options

- **Always-on service:** Full stack deployed into Kubernetes, running continuously for scheduled traffic evaluation (recommended for production optimization)
- **Ad-hoc run:** Temporary stack deployment for validation or proof-of-concept
- **Launchable / NGC / Helm:** NVIDIA provides a demo and Helm charts via NGC catalog for scalable deployment

***

## Quickstart (Conceptual Steps)

1. Prepare logs: Ensure schema compliance; tag each request with `workload_id` and `client_id`
2. Provision infrastructure: Elasticsearch, MongoDB, Redis, GPU compute for NeMo microservices
3. Deploy Flywheel API \& workers: Start blueprint API and task workers
4. Load/stream traffic: Push traffic into log index or stream live traffic
5. Launch a job: Trigger Flywheel run for selected workload(s)
6. Inspect results: Grouped by NIM and experiment type; examine LLM-as-judge scores, latency, and cost
7. Human review \& promotion: Review promising models/adapters; promote if acceptable

***

## Operational Runbook \& Best Practices

- **Tagging discipline:** Use stable `workload_id` across code releases
- **PII \& privacy:** Remove PII, comply with legal requirements before training
- **Human-in-the-loop:** Human performs final checks before promotion
- **Evaluate across multiple axes:** Combine accuracy, latency, and cost for candidate selection
- **Scheduling cadence:** Choose frequency based on traffic volume and cost constraints

***

## Hardware \& Software Requirements

- **Infrastructure:** Kubernetes cluster or servers for Docker, Elasticsearch, MongoDB, Redis; GPUs for fine-tuning/inference
- **Software:** NeMo Microservices platform, Flywheel blueprint services, Helm charts, NGC packages

References:

- [GitHub](https://github.com/NVIDIA-AI-Blueprints/data-flywheel)
- [NVIDIA NGC Catalog](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/blueprint/helm-charts/nvidia-blueprint-data-flywheel)

***

## Real-world Results \& Caveats

- **Reported gains:** Major cost reductions possible; e.g., using fine-tuned llama-3.2-1b instead of llama-3.1-70b for tool-calling tasks, with similar accuracy
- **Caveats:** Flywheel works best for narrow/structured production tasks, less effective for broad/open-domain tasks. Data quality is critical; noisy logs hinder effectiveness

***

## Security, Compliance \& Ethics

- Remove/mask PII before training unless legally permitted
- Maintain audit trails for datasets, experiments, promotions
- Review candidate models for unexpected behavior/bias before deployment

***

## Customizations \& Extensibility

- **Alternate log stores:** Adapt connectors for S3, BigQuery, etc.
- **Different fine-tuning strategies:** Extend Customizer for other methods (full-fine-tune, quantization+distillation, etc.)
- **Alternative evaluations:** Add ground-truth test sets, human-label pipelines for high-risk workloads

***

## Step-by-Step Adoption Plan

1. **Pilot:** Select a small, well-scoped workload and collect 1–2 weeks of traffic
2. **Run PoC:** Deploy blueprint in ad-hoc mode; analyze outputs
3. **Assess:** Evaluate models/adapters for accuracy, latency, cost; conduct safety checks
4. **Iterate:** Adjust datasets, experiment parameters, evaluation thresholds
5. **Scale:** Move to always-on deployment for broader workloads with governance and privacy safeguards

***

<div style="text-align: center">⁂</div>

[^1]: NVIDIA-Data-Flywheel.docx

