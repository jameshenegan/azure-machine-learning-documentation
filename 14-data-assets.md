# Create and manage data assets

https://learn.microsoft.com/en-us/azure/machine-learning/how-to-create-data-assets

### **Detailed Summary: Create and Manage Data Assets in Azure Machine Learning**

Azure Machine Learning **data assets** provide a structured and versioned way to manage datasets, ensuring reproducibility, auditability, and ease of access in machine learning workflows. This article details how to **create, manage, version, and organize data assets**.

---

## **1. Overview of Data Assets**

Data assets act as **bookmarks** for frequently used data stored in Azure, reducing the need to reference long storage paths manually.

### **Key Benefits of Data Assets**

1. **Versioning:** Each data asset version is immutable, ensuring stable datasets for experiments.
2. **Reproducibility:** Immutable versions allow consistent ML job executions.
3. **Auditability:** Tracks dataset changes, identifying who modified the dataset and when.
4. **Lineage Tracking:** Provides visibility into which jobs and pipelines use specific datasets.
5. **Ease-of-Use:** Simplifies dataset referencing with **friendly names** instead of long URIs.

> **Tip:** **Datastore URIs** can be used directly to access data in interactive sessions (e.g., Jupyter notebooks), **without creating a data asset**.

---

## **2. Prerequisites for Creating Data Assets**

To work with data assets, you need:

- **An Azure subscription**.
- **An Azure Machine Learning workspace**.
- **Azure ML CLI (v2) or Python SDK (v2)**.

---

## **3. Types of Data Assets**

Azure ML supports three **data asset types**, each suited for different use cases.

| **Type**   | **API Reference** | **Use Case**                                                               |
| ---------- | ----------------- | -------------------------------------------------------------------------- |
| **File**   | `uri_file`        | A single file (CSV, JSON, etc.).                                           |
| **Folder** | `uri_folder`      | A directory containing multiple files (e.g., images, text files, CSVs).    |
| **Table**  | `mltable`         | Structured tabular data that supports schema changes and AutoML workflows. |

> **Note:** **Embedded newlines in CSV files** can cause alignment issues unless registered as an **MLTable** with the `support_multi_line` parameter enabled.

---

## **4. Creating Data Assets**

Data assets are **defined using YAML configuration files** and can be created via **Azure CLI, Python SDK, or Azure ML Studio**.

### **4.1 Creating a File-Based Data Asset**

A **file (`uri_file`)** data asset represents a **single file**.

#### **Steps**

1. Create a **YAML file**:

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/data.schema.json
type: uri_file
name: my_csv_file
version: 1
description: Sample CSV file
path: wasbs://mycontainer@storageaccount.blob.core.windows.net/data/sample.csv
```

2. Run the CLI command:

```bash
az ml data create -f my_file_asset.yml
```

### **4.2 Creating a Folder-Based Data Asset**

A **folder (`uri_folder`)** data asset represents a **directory of files**.

#### **Steps**

1. Create a **YAML file**:

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/data.schema.json
type: uri_folder
name: my_image_folder
version: 1
description: Collection of images for deep learning
path: abfss://filesystem@storageaccount.dfs.core.windows.net/data/images/
```

2. Run the CLI command:

```bash
az ml data create -f my_folder_asset.yml
```

### **4.3 Creating a Table-Based Data Asset**

An **MLTable (`mltable`)** represents **structured tabular data**.

#### **Steps**

1. Create an `MLTable` file:

```yaml
paths:
  - file: wasbs://data@azuremlexampledata.blob.core.windows.net/titanic.csv
transformations:
  - read_delimited:
      delimiter: ","
      infer_column_types: true
      header: all_files_same_headers
```

2. Run the CLI command:

```bash
az ml data create --path ./data --name my_table_asset --version 1 --type mltable
```

> **Caution:** The `MLTable` file **must** be named exactly `MLTable`, **not** `MLTable.yaml`.

---

## **5. Creating Data Assets from Job Outputs**

Azure ML jobs can **generate data assets automatically** by specifying the output name in the job specification.

### **Example: Creating a Data Asset from a Job**

1. Define a **YAML job file**:

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/commandJob.schema.json
type: command
command: cp ${{inputs.input_data}} ${{outputs.output_data}}
compute: azureml:cpu-cluster
environment: azureml://registries/azureml/environments/sklearn-1.1/versions/4
inputs:
  input_data:
    mode: ro_mount
    path: wasbs://data@azuremlexampledata.blob.core.windows.net/titanic.csv
    type: uri_file
outputs:
  output_data:
    mode: rw_mount
    path: azureml://datastores/workspaceblobstore/paths/output/titanic.csv
    type: uri_file
    name: job_output_data_asset
```

2. Submit the job:

```bash
az ml job create --file my_job.yml
```

---

## **6. Managing Data Assets**

### **6.1 Data Asset Deletion**

Data assets **cannot be deleted** to maintain **lineage, reproducibility, and auditability**.

| **Reason for Deletion** | **Alternative Solution**                                             |
| ----------------------- | -------------------------------------------------------------------- |
| **Wrong name**          | **Archive** the data asset.                                          |
| **Obsolete data**       | **Archive** the data asset.                                          |
| **Incorrect path**      | Create a **new version** with the correct path.                      |
| **Incorrect type**      | (1) Archive the asset, (2) Create a new asset with the correct type. |

### **6.2 Archiving Data Assets**

- Archive **all versions**:
  ```bash
  az ml data archive --name my_data_asset
  ```
- Archive a **specific version**:
  ```bash
  az ml data archive --name my_data_asset --version 1
  ```

### **6.3 Restoring Archived Data Assets**

- Restore **all versions**:
  ```bash
  az ml data restore --name my_data_asset
  ```
- Restore a **specific version**:
  ```bash
  az ml data restore --name my_data_asset --version 1
  ```

---

## **7. Data Lineage in Azure ML**

Azure ML tracks **data lineage**, showing how datasets are consumed across **jobs and pipelines**.

- **Visual Lineage Tracking:** View dataset origins and transformations in **Azure ML Studio**.
- **Root Cause Analysis:** Identify failures in ML pipelines.

> **Screenshot Example:** Studio UI → **Data** → Select Data Asset → **View Jobs Consuming the Asset**.

---

## **8. Data Asset Tagging**

Tagging provides **additional metadata** for classification and governance.

### **Examples of Data Asset Tags**

| **Tag**              | **Use Case**                               |
| -------------------- | ------------------------------------------ |
| `medallion:gold`     | Data quality (e.g., gold, silver, bronze). |
| `sensitivity:PII`    | Identifying sensitive personal data.       |
| `RAI_audit:approved` | Responsible AI compliance.                 |

### **Adding Tags**

- During **creation**:
  ```yaml
  tags:
    data_quality: gold
    sensitivity: PII
  ```
- Updating an **existing asset**:
  ```bash
  az ml data update --name my_data_asset --version 1 --set tags.sensitivity=PII
  ```

---

## **9. Versioning Best Practices**

### **Time-Based Versioning**

Organizing datasets by **time-based folders** simplifies versioning.

#### **Example: Structuring Image Data by Week**

```
/myimages
 ├── year=2023
 │   ├── week1
 │   ├── week2
```

To create a versioned data asset:

```python
from azure.ai.ml.entities import Data
my_data = Data(
    path="abfss://filesystem@storageaccount.dfs.core.windows.net/myimages/year=2023/week1/",
    type="mltable",
    name="myimages",
    version="20230101"
)
```

> **Best Practice:** Use **MLTable** for structured versioning.

---

## **10. Key Takeaways**

- **Data assets enable versioning, reproducibility, and lineage tracking**.
- **Azure ML supports three asset types**: **File, Folder, Table**.
- **Data assets cannot be deleted** but can be **archived or updated**.
- **Data lineage tracking helps debug and analyze model workflows**.
- **Tagging enables metadata-based organization and governance**.

By **leveraging Azure ML data assets**, teams can ensure **efficient dataset management, seamless ML experimentation, and traceable model development**. 🚀
