

***

# NVIDIA Data Flywheel

## What the Data Flywheel Is

A **Data Flywheel** is an automated continuous workflow that uses production traffic (prompt/response logs, user feedback) to create evaluation and fine-tuning datasets, run experiments (different models, prompts, few-shot or fine-tuned variants), evaluate them, and surface promising smaller or cheaper models for human review and production promotion. The main goal is to reduce latency and inference cost while maintaining accuracy targets.
[GitHub](https://github.com/NVIDIA-AI-Blueprints/data-flywheel) -  [NVIDIA Developer](https://developer.nvidia.com/blog/build-efficient-ai-agents-through-model-distillation-with-nvidias-data-flywheel-blueprint/?utm_source=chatgpt.com)

***

## High-level Flow

- Production app emits logs (prompt, response, metadata: workload_id, client_id)
- Logs are centralized (Elasticsearch or other log store)
- Flywheel pulls logs → creates stratified eval and fine-tune datasets → runs candidate experiments (across NIMs, LoRA, few-shot, etc.)
- Results evaluated using LLM-as-judge metrics and other evaluations
- Flywheel surfaces promising candidates for human inspection and possible promotion
[GitHub](https://github.com/NVIDIA-AI-Blueprints/data-flywheel)

***

## Core Components

- **Log Store:** Central repository for prompts/responses (usually Elasticsearch)
- **NeMo Datastore:** Curated datasets from logs
- **NeMo Customizer:** Runs fine-tuning workflows (e.g., LoRA)
- **NeMo Evaluator:** Automated evaluations (LLM-as-judge metrics)
- **NeMo Deployment Manager/NIMs:** Runs inference microservices for evaluation and benchmark
- **Orchestrator/Flywheel API:** Control plane for scheduling, tracking dataset creation, experiments, results
- **Auxiliary Infrastructure:** MongoDB (metadata), Redis (queue/caching), Kubernetes or Docker
[GitHub](https://github.com/NVIDIA-AI-Blueprints/data-flywheel)

***

## Deployment Guide

### 1. Quick Summary

Provision a Kubernetes cluster with GPUs, install NVIDIA prerequisites (CUDA, toolkit), install NeMo/NIM operator and microservices, then deploy the Data Flywheel blueprint (Helm or manifests).
[GitHub](https://github.com/NVIDIA-AI-Blueprints/data-flywheel?utm_source=chatgpt.com) -  [NVIDIA Developer](https://developer.nvidia.com/blog/maximize-ai-agent-performance-with-data-flywheels-using-nvidia-nemo-microservices/?utm_source=chatgpt.com)

***

### 2. Prerequisites (Hardware/Platform)

- Kubernetes cluster (admin access; multi-node for prod, single-node for PoC)
- GPUs: NVIDIA data-center GPU (A100/H100 recommended)
- Disk: at least 200 GB free (microservices + datasets/artifacts)
- Linux hosts, Docker/containerd, NVIDIA drivers, Container Toolkit, CUDA runtime
- kubectl, helm, git, ansible (optional)
[NVIDIA Docs](https://docs.nvidia.com/nim-operator/latest/deploy-nemo-microservices.html?utm_source=chatgpt.com)

***

### 3. Accounts \& Keys

- NVIDIA NGC/API Catalog credentials (for blueprints, Helm charts)
- (Optional) Cloud provider credentials (if using cloud VMs for GPUs)
- Model registry/external API credentials (S3, artifact store)
[NVIDIA NGC Catalog](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/blueprint/helm-charts/nvidia-blueprint-data-flywheel?utm_source=chatgpt.com)

***

### 4. Prepare Cluster

1. Install NVIDIA driver and Container Toolkit (OS-specific)
2. Prepare Kubernetes namespaces \& RBAC:

```
kubectl create namespace nemo
kubectl create namespace data-flywheel
```

3. (Optional) Install ingress controller (NGINX/Contour) and storage class for PVCs
4. Ensure kubectl has cluster-admin access
[NVIDIA Docs](https://docs.nvidia.com/nim-operator/latest/deploy-nemo-microservices.html?utm_source=chatgpt.com)

***

### 5. Install NeMo/NIM Operator

- Clone NIM operator repo and checkout recommended tag:

```
git clone https://github.com/NVIDIA/k8s-nim-operator.git
cd k8s-nim-operator
git checkout v2.0.2  # Use doc-recommended version
```

- Run Ansible playbook/apply CRDs to install services:

```
kubectl apply -n nemo -f k8s-nim-operator/config/samples/nemo/latest/
```

- Follow NeMo prerequisites; verify pods are Running
[NVIDIA Docs](https://docs.nvidia.com/nim-operator/latest/deploy-nemo-microservices.html?utm_source=chatgpt.com)

***

### 6. Deploy Data Flywheel Blueprint

#### Option A: Helm Chart (Production)

- Get Data Flywheel Helm package from NGC (NGC credentials needed)
- Example install:

```
helm repo add nvidia-blueprints https://helm.ngc.nvidia.com/nvidia/blueprint
helm repo update
helm install data-flywheel nvidia-blueprints/nvidia-blueprint-data-flywheel \
  -n data-flywheel -f your-values.yaml
```

- Customize `your-values.yaml` for secrets, backends, resources
[NVIDIA NGC Catalog](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/blueprint/helm-charts/nvidia-blueprint-data-flywheel?utm_source=chatgpt.com)


#### Option B: GitHub Blueprint (Dev/Test)

- Clone Data Flywheel repo:

```
git clone https://github.com/NVIDIA-AI-Blueprints/data-flywheel.git
cd data-flywheel
```

- Follow README and quickstart, tailor config files; deploy with `kubectl apply -f` or `helm install`

[GitHub](https://github.com/NVIDIA-AI-Blueprints/data-flywheel?utm_source=chatgpt.com)

***

### 7. Configure Secrets \& Storage

- Create Kubernetes secrets for NGC keys, storage, DB passwords, e.g.:

```
kubectl create secret generic ngc-api-key -n data-flywheel --from-literal=key="${NGC_KEY}"
kubectl create secret generic s3-creds -n data-flywheel --from-literal=AWS_ACCESS_KEY_ID=... --from-literal=AWS_SECRET_ACCESS_KEY=...
```

- Configure persistent volumes for Elasticsearch, MongoDB, artifacts (usually automated by Helm if storageClass is set)

***

### 8. Post-deployment Verification

- Check pods/services

```
kubectl get pods -n data-flywheel
kubectl get svc -n data-flywheel
```

- Check logs for Orchestrator, Customizer, Evaluator, Datastore
- Use dashboards/API endpoints to confirm jobs/microservices
[NVIDIA Developer](https://developer.nvidia.com/blog/build-efficient-ai-agents-through-model-distillation-with-nvidias-data-flywheel-blueprint/?utm_source=chatgpt.com)

***

### 9. Tutorial / Experiment (PoC)

- NVIDIA provides Jupyter notebooks showing the loop (ingest → customize → eval → promote)
- Typical flow:
    - Ingest sample logs
    - Trigger customize job (LoRA fine-tune)
    - Trigger eval job (LLM-as-judge scoring)
    - Promote best candidate to deployment
- Verify model via NIM for inference
[NVIDIA Docs](https://docs.nvidia.com/nim-operator/latest/deploy-nemo-microservices.html?utm_source=chatgpt.com) -  [NVIDIA Developer](https://developer.nvidia.com/blog/new-video-build-self-improving-ai-agents-with-the-nvidia-data-flywheel-blueprint/?utm_source=chatgpt.com)

***

### 10. Observability \& Monitoring

- Use Prometheus + Grafana for metrics
- ELK/OpenSearch for logs, Elasticsearch for analytics
- Use Flywheel Orchestrator UI for job runs, metrics, evaluations
[NVIDIA NGC Catalog](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/blueprint/helm-charts/nvidia-blueprint-data-flywheel?utm_source=chatgpt.com)

***

### 11. Hardening \& Production Checklist

- Use secrets managers (Vault, K8s sealed secrets)
- Network policies, RBAC least privilege
- Resource limits and HPA for workers
- Backup ES, MongoDB, object storage for datasets/artifacts
- Implement NeMo Guardrails for safety and policy enforcement
[NVIDIA Developer](https://developer.nvidia.com/blog/maximize-ai-agent-performance-with-data-flywheels-using-nvidia-nemo-microservices/?utm_source=chatgpt.com)

***

### 12. Common Troubleshooting

- Pod CrashLoopBackOff: check logs and GPU/container runtime compatibility
- NeMo microservices DB errors: check service names, PVCs, secrets
- Jobs stuck: verify orchestrator-worker connectivity, pod resources
- Helm install fails with ClusterRole exists: clean up previous installs, use `helm uninstall` or unique release name
[GitHub](https://github.com/NVIDIA-AI-Blueprints/data-flywheel?utm_source=chatgpt.com) -  [NVIDIA Docs](https://docs.nvidia.com/nim-operator/latest/deploy-nemo-microservices.html?utm_source=chatgpt.com)

***

### 13. Useful Commands

```bash
# Create namespaces
kubectl create ns nemo data-flywheel

# Check pods
kubectl get pods -n data-flywheel -o wide

# Follow logs
kubectl logs -n data-flywheel -l app=flywheel-orchestrator -f

# Apply NeMo microservices
kubectl apply -n nemo -f k8s-nim-operator/config/samples/nemo/latest/
```

***

### 15. Primary References

- [NVIDIA Data Flywheel GitHub blueprint](https://github.com/NVIDIA-AI-Blueprints/data-flywheel?utm_source=chatgpt.com)
- [NeMo microservices Quickstart/Deploy docs](https://docs.nvidia.com/nim-operator/latest/deploy-nemo-microservices.html?utm_source=chatgpt.com)
- [NVIDIA developer blog: Maximize AI Agent Performance](https://developer.nvidia.com/blog/maximize-ai-agent-performance-with-data-flywheels-using-nvidia-nemo-microservices/?utm_source=chatgpt.com)
- [NGC Helm chart page (production)](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/blueprint/helm-charts/nvidia-blueprint-data-flywheel?utm_source=chatgpt.com)

---

<div style="text-align: center">⁂</div>

[^1]: NVIDIA-Data-Flywheel.docx

