# Data concepts in Azure Machine Learning

https://learn.microsoft.com/en-us/azure/machine-learning/concept-data

### **Detailed Summary of Data Concepts in Azure Machine Learning**

Azure Machine Learning (Azure ML) provides various **data management features** that allow users to efficiently import, store, and reference data from local or cloud-based storage. This article explains key **Azure ML data concepts**, including **datastores, data types, URIs, data runtime capabilities, and data assets**.

---

## **1. Datastore**

A **datastore** in Azure ML acts as a reference to an existing Azure storage account. It serves as an **intermediary** between Azure ML and various storage solutions, offering:

- A **consistent API** for interacting with multiple storage types (Blob, Files, ADLS).
- Simplified **team collaboration** by enabling easy discovery of shared data.
- **Secure storage credentials**, eliminating the need to hardcode sensitive connection details in scripts.

### **Authentication Methods for Datastores**

Azure ML datastores support **two authentication mechanisms**:

1. **Credential-based authentication**:
   - Uses **Service Principal, Shared Access Signature (SAS) tokens, or account keys**.
   - **Reader access** is required to retrieve credentials.
2. **Identity-based authentication**:
   - Uses **Microsoft Entra ID** (formerly Azure AD) or **Managed Identity**.

### **Supported Azure Storage Services for Datastores**

| **Storage Type**         | **Credential-Based** | **Identity-Based** |
| ------------------------ | -------------------- | ------------------ |
| **Azure Blob Container** | ✅ Yes               | ✅ Yes             |
| **Azure File Share**     | ✅ Yes               | ❌ No              |
| **Azure Data Lake Gen1** | ✅ Yes               | ✅ Yes             |
| **Azure Data Lake Gen2** | ✅ Yes               | ✅ Yes             |

### **Default Datastores in Azure ML**

Every Azure ML workspace has a **default storage account** with the following datastores:

| **Datastore Name**            | **Storage Type** | **Description**                                                   |
| ----------------------------- | ---------------- | ----------------------------------------------------------------- |
| **workspaceblobstore**        | Blob Container   | Stores **data uploads, job snapshots, and pipeline data caches**. |
| **workspaceworkingdirectory** | File Share       | Stores **notebooks, compute instances, and prompt flow data**.    |
| **workspacefilestore**        | File Share       | Serves as **an alternative container for data uploads**.          |
| **workspaceartifactstore**    | Blob Container   | Stores **models, metrics, and ML components**.                    |

---

## **2. Data Types in Azure Machine Learning**

Azure ML jobs **consume and produce data** in one of three main formats: **Files, Folders, and Tables**.

| **Type**   | **V2 API**   | **V1 API**       | **Common Use Cases**                                                                       | **API Differences**                                                                                                                   |
| ---------- | ------------ | ---------------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------- |
| **File**   | `uri_file`   | `FileDataset`    | Used for **reading/writing individual files** (any format).                                | V2 API directly maps files (no need for `os.path.join`).                                                                              |
| **Folder** | `uri_folder` | `FileDataset`    | Used for **image/text/audio/video datasets** or **structured data in CSV/Parquet format**. | In V2 API, folders are directly mapped instead of using a sampling engine.                                                            |
| **Table**  | `mltable`    | `TabularDataset` | Used for **complex schema datasets or AutoML workflows**.                                  | V2 API stores **data blueprints in storage** (not just in the Azure ML workspace), making it usable in **local and on-premise jobs**. |

---

## **3. URI in Azure ML**

A **Uniform Resource Identifier (URI)** is a reference to a **storage location** that can be **local, cloud-based, or public**.

### **Examples of URIs for Different Storage Types**

| **Storage Type**          | **URI Format Example**                                                          |
| ------------------------- | ------------------------------------------------------------------------------- |
| **Azure ML Datastore**    | `azureml://datastores/<data_store>/paths/<folder>/<file>.parquet`               |
| **Local Machine**         | `./home/username/data/my_data`                                                  |
| **Public HTTP(S) Server** | `https://raw.githubusercontent.com/pandas-dev/pandas/main/doc/data/titanic.csv` |
| **Azure Blob Storage**    | `wasbs://<container>@<account>.blob.core.windows.net/<folder>/`                 |
| **Azure Data Lake Gen2**  | `abfss://<filesystem>@<account>.dfs.core.windows.net/<folder>/file.csv`         |

### **URI Authentication**

- Azure ML **maps URIs to the compute target filesystem**.
- **Identity-based authentication (Microsoft Entra ID)** is used by default.
- **Credential-based authentication** (e.g., Service Principal, SAS tokens) is also supported.

---

## **4. Data Input/Output Modes in Azure ML Jobs**

URIs can be used **as inputs or outputs** in Azure ML jobs. The following modes define how data is accessed:

| **Mode**                          | **Description**                                       | **Supports Input?** | **Supports Output?** |
| --------------------------------- | ----------------------------------------------------- | ------------------- | -------------------- |
| **Read-only mount (`ro_mount`)**  | Mounts a **read-only** storage location.              | ✅ Yes              | ❌ No                |
| **Read-write mount (`rw_mount`)** | Mounts a storage location with **read/write access**. | ✅ Yes              | ✅ Yes               |
| **Download (`download`)**         | **Downloads data** to the compute target.             | ✅ Yes              | ❌ No                |
| **Upload (`upload`)**             | **Uploads data** from the compute target to storage.  | ❌ No               | ✅ Yes               |

### **Job Input/Output Modes Summary**

| **Input/Output Type** | `upload` | `download` | `ro_mount` | `rw_mount` | `direct` |
| --------------------- | -------- | ---------- | ---------- | ---------- | -------- |
| **Input**             | ❌ No    | ✅ Yes     | ✅ Yes     | ❌ No      | ✅ Yes   |
| **Output**            | ✅ Yes   | ❌ No      | ❌ No      | ✅ Yes     | ❌ No    |

---

## **5. Data Runtime Capability in Azure ML**

Azure ML has a built-in **data runtime** optimized for **high-performance machine learning workloads**.

### **Key Features**

- **Rust-based architecture** → Faster execution, low memory usage.
- **Lightweight installation** → No external dependencies (e.g., no JVM).
- **Multi-process parallel data loading** → Faster training and inference.
- **Automatic data pre-fetching** → **Optimizes GPU utilization** for deep learning.
- **Seamless cloud authentication** → No need for manual authentication setup.

### **Data Runtime Functions**

1. **Manages data mounting, uploads, and downloads.**
2. **Maps URIs to the compute target filesystem.**
3. **Materializes tabular data (`mltable`) for use in Pandas/Spark.**

---

## **6. Data Assets in Azure ML**

A **Data Asset** in Azure ML functions like a **bookmark** for frequently used datasets.

### **Benefits of Data Assets**

- **Simplifies access to storage locations** (no need to remember long URIs).
- **Stores metadata** about datasets **without duplicating data**.
- **No extra storage costs** or risk of altering original data.

### **Data Asset Creation**

Data assets can reference data from:

- **Azure ML Datastores**
- **Azure Storage (Blob, ADLS)**
- **Public URLs**
- **Local files**

---

## **7. Key Takeaways**

1. **Datastores** serve as **references to Azure storage accounts** and support **credential-based & identity-based authentication**.
2. **Azure ML supports three data types**:
   - **File (`uri_file`)** – Single files.
   - **Folder (`uri_folder`)** – Multiple files in a directory.
   - **Table (`mltable`)** – Structured data tables.
3. **URIs define storage locations** and support authentication via **Microsoft Entra ID or credentials**.
4. **Data access modes** include:
   - **Mounting (`ro_mount`, `rw_mount`)**
   - **Downloading (`download`)**
   - **Uploading (`upload`)**
5. **Azure ML Data Runtime** optimizes performance using:
   - **Rust-based architecture**
   - **Parallel data loading**
   - **Automated GPU utilization**
6. **Data Assets act as bookmarks** for commonly used datasets, simplifying dataset management.

By using **Azure ML data concepts efficiently**, users can **streamline data access, improve model training speed, and ensure secure and reproducible workflows**.
