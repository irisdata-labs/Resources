

## 🧭 Components
> Assumes AWS as the cloud provider for example purposes

### 1. Compute & Memory
- Applies to: all Milvus services deployed via Kubernetes  
- Includes: Proxy, QueryNode, DataNode, IndexNode, Coord services  
- AWS Cost driver: EC2 instances  
- Influenced by:
  - Vector dimensions  
  - QPS  
  - Index type  
  - Replicas

### 2. Storage

| Type            | Used By                   | AWS Resource |
|-----------------|---------------------------|---------------|
| Object Storage  | Flushed segments, indexes | Amazon S3     |
| Block Storage   | Etcd, BookKeeper logs     | Amazon EBS    |

### 3. Messaging Layer (Pulsar)
- Pulsar Brokers + BookKeeper = EC2 + EBS  
- Data + WALs stored in EBS

### 4. Monitoring (Optional)
- **Datadog** for metrics/alerting (SaaS pricing)
- **CloudWatch** for AWS infrastructure/billing logs

---

## 📊 Performance Estimates & Resource Allocation

### Step 1: Vectorizer Service
- 2 CPU → ~44 QPS (based on 45–50ms per request)  
- Embedding latency (CPU): ~45ms/query  
- Model: `intfloat/e5-small-v2` (384-dim)

### Step 2: Milvus QueryNode
- QPS: 35–45  
- Index type: IVF_FLAT  
- Vector count: ~3M  
- Vector size: 384 × 4B = 1.5 KB  
- Total: ~4.5 GB + index overhead → Need ~8 GiB memory  
- CPU: ~2 vCPUs to support search load

---

## 💸 Resource Spec deployed on Kubernetes

| Component          | Type (example only)           | Spec                     | Qty | 
|-------------------|-----------------|---------------------------|-----|
| **Proxy**          | EC2 (m5.large)  | 1 vCPU / 1 GiB            | 1   |            
| **QueryNode**      | EC2 (m5.large)  | 2 vCPU / 8 GiB            | 1   | 
| **DataNode**       | EC2 (m5.large)  | 2 vCPU / 8 GiB            | 1   | 
| **IndexNode**      | EC2 (m5.xlarge) | 4 vCPU / 16 GiB           | 1   | 
| **etcd**           | EC2 (t3.small) + EBS | 0.5 vCPU / 2 GiB + 10 GiB | 1 |
| **BookKeeper**     | EC2 (m5.large) + EBS | 2 vCPU / 8 GiB + 100 GiB | 3 | 
| **S3 Storage**     | S3 Standard     | ~50 GB                    | —   |
| **Index temp (EBS)** | gp3          | 40 GiB                    | 1   | 
| **CloudWatch**     | Optional logs   | —                         | —   | 
| **Datadog**        | Optional metrics| Up to 200 metrics         | —   | 

> EKS control plane — shared across multiple services running in the same Kubernetes cluster
