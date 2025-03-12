# How Azure Machine Learning works: resources and assets

https://learn.microsoft.com/en-us/azure/machine-learning/concept-azure-machine-learning-v2

### **Detailed Summary: How Azure Machine Learning Works – Resources and Assets**

Azure Machine Learning (Azure ML) is a **comprehensive platform** that provides a set of **resources** and **assets** to build, train, and deploy machine learning models. This article explains the **core components of Azure ML**, including workspaces, compute resources, datastores, and model management.

---

## **1. Overview: Resources vs. Assets**

### **🔹 Resources** (Infrastructure for ML workflows)

These are **setup or infrastructural elements** required to run machine learning tasks:

- **Workspace** – The central hub for managing all ML-related resources.
- **Compute** – Resources used for training and inference.
- **Datastore** – Secure storage for data used in ML workflows.

### **🔹 Assets** (Versioned & Registered ML Elements)

These are **created and versioned** within a workspace:

- **Model** – A trained ML model stored in Azure ML.
- **Environment** – The software environment defining dependencies.
- **Data** – The dataset used in ML experiments.
- **Component** – A reusable, self-contained unit in an ML pipeline.

---

## **2. Prerequisites**

Before using Azure ML, you need to:

1. **Install Azure ML Python SDK v2**.
2. **Authenticate and connect** to Azure ML.

### **🔹 Connect to Azure ML Using Python**

```python
from azure.ai.ml import MLClient
from azure.ai.ml.entities import Workspace
from azure.identity import DefaultAzureCredential

# Authentication
subscription_id = "<SUBSCRIPTION_ID>"
resource_group = "<RESOURCE_GROUP>"

# Connect to subscription
ml_client = MLClient(DefaultAzureCredential(), subscription_id, resource_group)

# Connect to workspace
workspace_name = "<WORKSPACE_NAME>"
ml_client = MLClient(DefaultAzureCredential(), subscription_id, resource_group, workspace_name)
```

### **🔹 Authenticate via Azure CLI**

```bash
az login
az account set -s "<YOUR_SUBSCRIPTION_NAME_OR_ID>"
```

---

## **3. Workspace – The Central ML Resource**

A **workspace** is the **top-level Azure ML resource**, acting as the **hub** for managing:
✔ **Jobs & experiments**  
✔ **Compute resources**  
✔ **Data assets**  
✔ **Trained models**

### **🔹 Create a Workspace Using Python**

```python
from azure.ai.ml.entities import Workspace

ws = Workspace(
    name="my_workspace",
    location="eastus",
    display_name="My workspace",
    description="Example workspace",
    tags={"purpose": "demo"},
)

ml_client.workspaces.begin_create(ws)
```

### **🔹 Create a Workspace Using CLI**

```bash
az ml workspace create --file my_workspace.yml
```

👉 _For more details: [Manage Azure ML workspaces](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-manage-workspace)_

---

## **4. Compute – Running ML Workloads**

Compute is used to **run jobs and host models**. Azure ML offers various compute options:

| **Compute Type**       | **Description**                                                |
| ---------------------- | -------------------------------------------------------------- |
| **Compute Instance**   | A fully managed **cloud-based** development environment.       |
| **Compute Cluster**    | A **scalable** set of CPU/GPU nodes for distributed training.  |
| **Serverless Compute** | A **dynamically allocated** compute resource.                  |
| **Inference Cluster**  | A **Kubernetes-based** cluster for model deployment.           |
| **Attached Compute**   | **Bring your own** compute (e.g., on-prem or other cloud VMs). |

### **🔹 Create a Compute Cluster Using Python**

```python
from azure.ai.ml.entities import AmlCompute

cluster = AmlCompute(
    name="compute-cluster",
    type="amlcompute",
    size="STANDARD_DS3_v2",
    min_instances=0,
    max_instances=2,
    idle_time_before_scale_down=120,
)

ml_client.begin_create_or_update(cluster)
```

### **🔹 Create a Compute Cluster Using CLI**

```bash
az ml compute create --file my_compute.yml
```

👉 _For more details: [Create an Azure ML compute cluster](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-create-manage-compute-clusters)_

---

## **5. Datastore – Secure Data Management**

A **datastore** securely **connects Azure ML to data storage** (Azure Blob, Data Lake, etc.).

### **🔹 Supported Storage Options**

- **Azure Blob Container**
- **Azure File Share**
- **Azure Data Lake Storage (ADLS)**
- **Azure Data Lake Gen2**

### **🔹 Create a Datastore Using Python**

```python
from azure.ai.ml.entities import AzureBlobDatastore

blob_datastore = AzureBlobDatastore(
    name="blob_example",
    description="Datastore pointing to Azure Blob Storage",
    account_name="my_blob_storage",
    container_name="data-container",
    credentials={"account_key": "XXXxxxXXXxXXXXxxXXXXX"},
)

ml_client.create_or_update(blob_datastore)
```

### **🔹 Create a Datastore Using CLI**

```bash
az ml datastore create --file my_datastore.yml
```

👉 _For more details: [Create and manage Azure ML datastores](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-access-data)_

---

## **6. Model – Storing Trained ML Models**

A **model** in Azure ML consists of:
✔ **Binary files** representing the trained model.  
✔ **Metadata** about model parameters & performance.  
✔ **Version control** for tracking updates.

### **🔹 Supported Model Formats**

- **Custom Model**
- **MLflow Model**
- **Triton Model**

👉 _For more details: [Work with models in Azure Machine Learning](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-manage-models)_

---

## **7. Environment – Reproducible ML Execution**

An **environment** defines the **runtime settings** for ML jobs.

### **🔹 Types of Environments**

| **Type**    | **Description**                                             |
| ----------- | ----------------------------------------------------------- |
| **Curated** | Prebuilt Azure ML environments (e.g., TensorFlow, PyTorch). |
| **Custom**  | User-defined environments using Conda/Docker.               |

### **🔹 Creating a Custom Environment**

```python
from azure.ai.ml.entities import Environment

custom_env = Environment(
    name="custom-env",
    description="Custom environment for training",
    conda_file="environment.yml",
    image="mcr.microsoft.com/azureml/openmpi4.1.0-ubuntu20.04:latest",
)

ml_client.environments.create_or_update(custom_env)
```

👉 _For more details: [Manage environments in Azure ML](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-use-environments)_

---

## **8. Data – Managing Datasets**

Azure ML supports:

- **URIs** (cloud/local storage locations)
  - `uri_folder`
  - `uri_file`
- **Tables** (`mltable`) – Tabular data abstraction.
- **Primitives** (`string`, `boolean`, `number`).

🔹 **Recommended Approach**: Use **URIs** for accessing data in storage.

👉 _For more details: [Create and manage data assets](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-create-manage-data-assets)_

---

## **9. Component – Building Blocks of Pipelines**

A **component** is a **self-contained piece of code** that performs a specific ML task.

✔ **Examples**:

- Data preprocessing
- Model training
- Model scoring

### **🔹 Example: Define a Component**

```python
from azure.ai.ml import command
from azure.ai.ml import Input, Output

data_prep_component = command(
    name="data_prep",
    display_name="Data Preparation",
    inputs={"data": Input(type="uri_folder")},
    outputs={"cleaned_data": Output(type="uri_folder")},
    command="python data_prep.py --data ${{inputs.data}} --cleaned_data ${{outputs.cleaned_data}}",
    environment="custom-env@latest",
)
```

👉 _For more details: [Build ML pipelines in Azure](https://learn.microsoft.com/en-us/azure/machine-learning/concept-pipelines)_

---

## **Conclusion**

Azure Machine Learning provides a **structured approach** to ML workflows by organizing **resources (infrastructure)** and **assets (models, environments, data, etc.)**. It enables **scalable, reproducible, and efficient ML model development and deployment**.

**🚀 Next Steps:**

- [Upgrade from SDK v1 to v2](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-upgrade-v1-v2)
- [Train models using CLI & SDK v2](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-train-models-cli-v2)
