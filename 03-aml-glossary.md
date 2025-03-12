# Azure Machine Learning glossary

https://learn.microsoft.com/en-us/azure/machine-learning/azure-machine-learning-glossary

### **Detailed Summary: Azure Machine Learning Glossary**

The **Azure Machine Learning glossary** provides definitions for key terms related to the Azure Machine Learning platform. It helps users understand essential concepts such as components, compute resources, data management, and machine learning environments.

---

## **Key Terminology in Azure Machine Learning**

### **1. Component**

A **component** in Azure Machine Learning is a **self-contained unit of code** that performs a **specific step in a machine learning pipeline**. It is analogous to a function, with:

- A **name** and **parameters**.
- **Inputs** and **outputs**.
- Examples of tasks: **data processing, model training, model scoring**.
- **Components** help **modularize and reuse** ML logic across pipelines.

---

### **2. Compute**

**Compute** refers to the resources where machine learning jobs run or where models are hosted for inferencing.

#### **Types of Compute in Azure ML:**

1. **Compute Cluster**

   - A managed **CPU or GPU cluster** in the cloud.
   - Ideal for large-scale training and inference tasks.
   - **Alternative**: **Serverless compute** offloads lifecycle management.

2. **Compute Instance**

   - A **fully managed development environment** in the cloud.
   - Similar to a **virtual machine (VM)**, used for **training and testing**.

3. **Kubernetes Cluster (AKS)**

   - Used to **deploy ML models** via **Azure Kubernetes Service (AKS)**.
   - Can create an **AKS cluster** from an Azure ML workspace or attach an existing one.

4. **Attached Compute**
   - Users can **connect their own compute resources** (on-premise or cloud-based) to Azure ML for training and inference.

---

### **3. Data**

Azure ML supports multiple **data types and storage locations**:

1. **Uniform Resource Identifiers (URIs)** – Storage locations in **local or cloud environments**:

   - `uri_folder` – Points to a **folder in storage**.
   - `uri_file` – Points to a **specific file** in storage.

2. **Tables** – Tabular datasets used for structured ML workflows:

   - `mltable` – A special data format for **AutoML and parallel jobs**.

3. **Primitives** – Simple data types:
   - `string`, `boolean`, `number`.

For most ML tasks, **URIs** (e.g., `uri_folder`) are used to reference cloud storage **mounted or downloaded** to compute nodes.

---

### **4. Datastore**

A **datastore** is a secure **connection point to data storage** on Azure. Instead of hardcoding connection details in scripts, users can register a datastore and access it easily.

#### **Supported Storage Services for Datastores:**

- **Azure Blob Storage**
- **Azure Files**
- **Azure Data Lake Storage** (Gen1 & Gen2)

Datastores simplify **data access and management** in Azure ML projects.

---

### **5. Environment**

A **Machine Learning environment** encapsulates the **software setup** needed for ML training and inference.

#### **Types of ML Environments:**

1. **Curated Environments**

   - **Predefined by Azure ML** and available **by default** in a workspace.
   - Contain commonly used **Python packages** for machine learning.
   - Offer **faster deployment**.

2. **Custom Environments**
   - Users can define their own environment by using:
     - **A custom Docker image**.
     - **A base Docker image** with a **conda YAML** for customization.
     - **A Docker build context**.

Environments **ensure reproducibility** and **consistency** in ML workflows.

---

### **6. Model**

A **machine learning model** in Azure ML consists of **binary files and metadata** representing a trained ML model.

#### **Supported Model Storage Formats:**

- **`custom_model`** – User-defined model storage format.
- **`mlflow_model`** – MLflow-tracked models.
- **`triton_model`** – NVIDIA Triton Inference Server model format.

Models can be stored **locally or remotely** in locations like:

- `https`
- `wasbs` (Azure Blob Storage)
- `azureml` (Azure Machine Learning storage)

Azure ML **tracks models with versioning** inside the **workspace**.

---

### **7. Workspace**

An **Azure Machine Learning workspace** is the **centralized resource** for ML projects.

#### **Key Functions of a Workspace:**

- Stores **history** of jobs (logs, metrics, output, and snapshots).
- **Manages assets**:
  - **Models**
  - **Environments**
  - **Data assets**
  - **Components**
- Provides references to:
  - **Datastores** (for data storage).
  - **Compute resources** (for training and inference).

The **workspace** enables collaboration, **experiment tracking**, and **project management** in Azure ML.

---

## **Conclusion**

The **Azure Machine Learning glossary** defines essential ML concepts in Azure, covering **compute resources, data management, environments, models, and workspaces**. Understanding these terms helps users efficiently navigate and leverage the **Azure ML ecosystem** for building, training, and deploying machine learning models.
