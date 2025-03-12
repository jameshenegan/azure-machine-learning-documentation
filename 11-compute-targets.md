# What are compute targets in Azure Machine Learning?

https://learn.microsoft.com/en-us/azure/machine-learning/concept-compute-target

### **Detailed Summary of Compute Targets in Azure Machine Learning**

Azure Machine Learning (Azure ML) provides various compute targets, which are designated computing environments where users can run training scripts or deploy machine learning models. These environments range from local machines to cloud-based resources, allowing flexibility and scalability without requiring code modifications.

---

## **1. Overview of Compute Targets**

Compute targets are categorized based on their purpose:

- **Training Compute Targets:** Used to run machine learning training jobs, supporting both single-node and distributed training.
- **Compute Targets for Inference:** Used for hosting trained models to serve predictions in real-time or batch mode.
- **Managed Compute Targets:** Azure ML-managed compute clusters, serverless compute, and compute instances optimized for machine learning workloads.
- **Unmanaged Compute Targets:** User-managed resources such as virtual machines (VMs), Kubernetes clusters, and external cloud services.

---

## **2. Training Compute Targets**

Training compute targets allow users to scale up model training by leveraging cloud-based resources. They can be:

- **Reused:** Once a compute target is attached to an Azure ML workspace, it can be used for multiple jobs.
- **Autoscaled:** Azure ML compute clusters automatically scale up/down based on the submitted jobs.

### **Supported Training Compute Targets**

| Compute Target                   | Automated ML         | ML Pipelines | Azure ML Designer |
| -------------------------------- | -------------------- | ------------ | ----------------- |
| **Azure ML Compute Cluster**     | Yes                  | Yes          | Yes               |
| **Azure ML Serverless Compute**  | Yes                  | Yes          | Yes               |
| **Azure ML Compute Instance**    | Yes (SDK)            | Yes          | Yes               |
| **Azure ML Kubernetes**          | No                   | Yes          | Yes               |
| **Remote VM**                    | Yes                  | Yes          | No                |
| **Apache Spark Pools (Preview)** | Yes (SDK local mode) | Yes          | No                |
| **Azure Databricks**             | Yes (SDK local mode) | Yes          | No                |
| **Azure Data Lake Analytics**    | No                   | Yes          | No                |
| **Azure HDInsight**              | No                   | Yes          | No                |
| **Azure Batch**                  | No                   | Yes          | No                |

#### **Key Considerations for Training Compute Targets**

- **Compute Instance Storage:** Has a **120GB OS disk**. Users must clear space manually if needed.
- **Azure Databricks:** Only supports local runs and ML pipelines, not remote training.
- **Distributed Training:** Achievable through Azure ML compute clusters and Kubernetes.

---

## **3. Compute Targets for Inference**

Azure ML deploys models inside **Docker containers**, which are hosted on compute targets optimized for inference.

### **Supported Inference Compute Targets**

| Compute Target          | Use Case                    | GPU Support | Description                                                                      |
| ----------------------- | --------------------------- | ----------- | -------------------------------------------------------------------------------- |
| **Azure ML Endpoints**  | Real-time & batch inference | Yes         | Fully managed compute for online and batch inference                             |
| **Azure ML Kubernetes** | Real-time & batch inference | Yes         | Supports inference workloads on cloud, on-premises, and edge Kubernetes clusters |

#### **Best Practices for Inference Compute**

- **Memory Allocation:** Start with **150% of the RAM** needed for the model.
- **Scaling:** First, **scale up (larger machines)**, then **scale out (more instances)**.

---

## **4. Managed Compute in Azure ML**

Managed compute resources are automatically provisioned, scaled, and optimized by Azure ML.

### **Managed Compute Types**

| Capability                | Compute Cluster | Compute Instance |
| ------------------------- | --------------- | ---------------- |
| Single/Multi-node cluster | ✓               | Single node      |
| Autoscaling               | ✓               | No               |
| Automatic job scheduling  | ✓               | ✓                |
| Supports CPU & GPU        | ✓               | ✓                |

### **Cost Optimization**

- **Compute Clusters:** Set **minimum nodes to 0** when idle.
- **Compute Instances:** Enable **idle shutdown** to prevent unnecessary costs.

---

## **5. Supported VM Series & Sizes**

Azure ML supports various VM types categorized based on their workload.

| VM Series                                                 | Category                 | Supported By                 |
| --------------------------------------------------------- | ------------------------ | ---------------------------- |
| DDSv4, Dv2, Dv3, DSv2, DSv3                               | General Purpose          | Compute Clusters & Instances |
| EAv4, Ev3, ESv3                                           | Memory Optimized         | Compute Clusters & Instances |
| FSv2, FX                                                  | Compute Optimized        | Compute Clusters             |
| H, HB, HBv2, HBv3, HC                                     | High Performance Compute | Compute Clusters & Instances |
| LSv2                                                      | Storage Optimized        | Compute Clusters & Instances |
| M                                                         | Memory Optimized         | Compute Clusters & Instances |
| NC, NCv2, NCv3, ND, NDv2, NV, NVv3, NCasT4_v3, ND_A100_v4 | GPU                      | Compute Clusters & Instances |

### **Deprecated VM Series**

| VM Series                                    | Retirement Date |
| -------------------------------------------- | --------------- |
| NC-series, NCv2-series, ND-series, NV-series | August 31, 2023 |
| Av1-series, HB-series                        | August 31, 2024 |

### **CUDA Compatibility**

For GPU-enabled compute targets, users must install the correct CUDA version:
| GPU Architecture | Azure VM Series | Supported CUDA Versions |
|------------------|----------------|-------------------------|
| Ampere | NDA100_v4 | 11.0+ |
| Turing | NCT4_v3 | 10.0+ |
| Volta | NCv3, NDv2 | 9.0+ |
| Pascal | NCv2, ND | 9.0+ |
| Maxwell | NV, NVv3 | 9.0+ |
| Kepler | NC, NC Promo | 9.0+ |

#### **Compute Isolation**

Certain VM sizes offer **isolated environments** for security and compliance needs.
| Isolated VM | Type |
|------------|------|
| Standard_M128ms | High-memory workload |
| Standard_F72s_v2 | Compute-optimized workload |
| Standard_NC24s_v3 | GPU-intensive workload |
| Standard_NC24rs_v3 | RDMA-enabled GPU workload |

---

## **6. Unmanaged Compute Targets**

Unlike managed compute, these require manual setup and maintenance.

### **Supported Unmanaged Compute Targets**

- **Remote VMs**
- **Azure HDInsight**
- **Azure Databricks**
- **Azure Data Lake Analytics**
- **Kubernetes**

These allow integration with **external compute environments**, but require users to handle their lifecycle, configurations, and performance optimizations.

---

## **7. Key Takeaways**

1. **Training Compute Targets** enable **scaling of ML workloads**, supporting both **single-node and distributed training**.
2. **Inference Compute Targets** use **Docker containers** to host models, with options for **real-time and batch inference**.
3. **Managed Compute (Azure ML Compute Clusters, Instances, Serverless)** simplifies resource provisioning and scaling.
4. **Unmanaged Compute (Remote VMs, Kubernetes, Databricks, etc.)** requires additional setup but offers flexibility.
5. **Cost Considerations:** Autoscaling clusters and enabling idle shutdown for instances can prevent **unnecessary costs**.
6. **VM Selection:** Some series are **deprecated**, and **CUDA versions must match the GPU architecture** for training jobs.
7. **Compute Isolation:** Dedicated VMs ensure **high security and regulatory compliance**.

By leveraging the right compute target in Azure Machine Learning, users can optimize performance, scalability, and cost efficiency for their ML workloads.
