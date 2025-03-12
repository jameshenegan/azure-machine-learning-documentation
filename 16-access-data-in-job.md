# Access data in a job

https://learn.microsoft.com/en-us/azure/machine-learning/how-to-read-write-data-v2

### **Detailed Summary: Access Data in an Azure Machine Learning Job**

Azure Machine Learning (Azure ML) allows users to efficiently access, read, and write data from Azure Storage within ML jobs. This article covers:

- **Reading and writing data within Azure ML jobs**
- **Different data access modes (mount vs. download)**
- **Authentication methods (User Identity & Managed Identity)**
- **Performance optimizations for handling large datasets**

---

## **1. Prerequisites**

- **Azure subscription** (Free or paid version).
- **Azure ML SDK for Python v2**.
- **An Azure ML workspace**.

---

## **2. Quickstart: Access Data in an Azure ML Job**

### **Read Data from Azure Storage in an ML Job**

Azure ML supports accessing data stored in:

- **Blob Storage**
- **Azure Data Lake Storage (ADLS)**
- **Azure ML Datastore**
- **Public URLs (HTTP/HTTPS)**

> **Authentication Options**

1. **User Identity** → Uses Microsoft Entra ID.
2. **Managed Identity** → Uses the compute target's managed identity.
3. **None** → Use credential-based authentication (SAS token, storage key).

### **Example: Read a CSV File in a Job**

```python
from azure.ai.ml import command, Input, MLClient
from azure.ai.ml.constants import AssetTypes, InputOutputModes
from azure.identity import DefaultAzureCredential

ml_client = MLClient(DefaultAzureCredential(), "<SUBSCRIPTION_ID>", "<RESOURCE_GROUP>", "<AML_WORKSPACE_NAME>")

path = "wasbs://data@azuremlexampledata.blob.core.windows.net/titanic.csv"

inputs = {
    "input_data": Input(type=AssetTypes.URI_FILE, path=path, mode=InputOutputModes.RO_MOUNT)
}

job = command(
    command="head ${{inputs.input_data}}",
    inputs=inputs,
    environment="azureml://registries/azureml/environments/sklearn-1.1/versions/4",
    compute="cpu-cluster",
)

ml_client.jobs.create_or_update(job)
```

---

## **3. Writing Data to Azure Storage**

Azure ML allows job outputs to be written to:

- **Azure ML Datastores**
- **Blob Storage**
- **ADLS**

### **Example: Write Data to Azure ML Datastore**

```python
from azure.ai.ml import command, Input, Output, MLClient
from azure.ai.ml.constants import AssetTypes, InputOutputModes
from azure.identity import DefaultAzureCredential

ml_client = MLClient(DefaultAzureCredential(), "<SUBSCRIPTION_ID>", "<RESOURCE_GROUP>", "<AML_WORKSPACE_NAME>")

input_path = "wasbs://data@azuremlexampledata.blob.core.windows.net/titanic.csv"
output_path = "azureml://datastores/workspaceblobstore/paths/quickstart-output/titanic.csv"

inputs = {
    "input_data": Input(type=AssetTypes.URI_FILE, path=input_path, mode=InputOutputModes.RO_MOUNT)
}

outputs = {
    "output_data": Output(type=AssetTypes.URI_FILE, path=output_path, mode=InputOutputModes.RW_MOUNT)
}

job = command(
    command="cp ${{inputs.input_data}} ${{outputs.output_data}}",
    inputs=inputs,
    outputs=outputs,
    environment="azureml://registries/azureml/environments/sklearn-1.1/versions/4",
    compute="cpu-cluster",
)

ml_client.jobs.create_or_update(job)
```

---

## **4. Azure ML Data Runtime**

Azure ML’s **data runtime** optimizes data loading with:

- **Rust-based architecture** → High performance and memory efficiency.
- **Parallel data loading** → Faster data transfers.
- **Prefetching** → Background CPU tasks to optimize GPU performance.
- **Seamless authentication** → Avoids explicit credential handling.

> **Best Practice:** Use the **Azure ML data runtime** instead of writing custom mounting or downloading logic.

---

## **5. Data Paths in Azure ML**

Azure ML jobs support various **path formats** for data inputs and outputs:

| **Storage Type**    | **Path Format**                                        |
| ------------------- | ------------------------------------------------------ |
| Local               | `./data/myfile.csv`                                    |
| HTTP                | `https://example.com/data.csv`                         |
| Azure Blob Storage  | `wasbs://container@account.blob.core.windows.net/path` |
| ADLS Gen2           | `abfss://filesystem@account.dfs.core.windows.net/path` |
| Azure ML Datastore  | `azureml://datastores/datastore_name/paths/path`       |
| Azure ML Data Asset | `azureml:my_data_asset:version`                        |

---

## **6. Data Access Modes**

| **Mode**                       | **Description**                 | **Best For**                                    |
| ------------------------------ | ------------------------------- | ----------------------------------------------- |
| `ro_mount`                     | Read-only mount                 | Large datasets, no need for modification        |
| `rw_mount`                     | Read-write mount                | Temporary modifications during training         |
| `download`                     | Downloads data before execution | Small datasets, repeated access                 |
| `upload`                       | Uploads data after execution    | Saving processed outputs                        |
| `eval_mount` / `eval_download` | Special for `mltable`           | Subsetting, shuffling, or multi-source datasets |
| `direct`                       | Read directly from URI          | Streaming scenarios                             |

### **Example: Mount vs. Download Performance**

| **Mode**     | **Pros**                             | **Cons**                      |
| ------------ | ------------------------------------ | ----------------------------- |
| **Download** | No network dependency after download | Needs disk space              |
| **Mount**    | No delay before training             | Slower for random file access |

> **Choosing Between Download and Mount**

- **Use Download** when the dataset **fits on disk** and will be read **multiple times**.
- **Use Mount** for **large datasets** that **don’t fit on local disk**.

---

## **7. Performance Optimization**

### **Optimizing Downloads**

| **Setting**                        | **Description**                       | **Default**     |
| ---------------------------------- | ------------------------------------- | --------------- |
| `RSLEX_DOWNLOADER_THREADS`         | Number of concurrent download threads | `CPU_CORES * 4` |
| `AZUREML_DATASET_HTTP_RETRY_COUNT` | Retry count for storage requests      | `7`             |

```python
env_var = {
    "RSLEX_DOWNLOADER_THREADS": 64,
    "AZUREML_DATASET_HTTP_RETRY_COUNT": 10
}

job = command(environment_variables=env_var)
```

### **Optimizing Mount Performance**

| **Setting**                         | **Description**                | **Default**     |
| ----------------------------------- | ------------------------------ | --------------- |
| `DATASET_MOUNT_ATTRIBUTE_CACHE_TTL` | Cache expiration for metadata  | No expiration   |
| `DATASET_MOUNT_READ_BLOCK_SIZE`     | Block size for streaming reads | `2MB`           |
| `DATASET_MOUNT_READ_THREADS`        | Number of prefetching threads  | `CPU_CORES * 4` |

---

## **8. Handling Large Data**

> **Strategies for Large Files**

- **Sequential Reads** → Use **block-based caching**.
- **Random Reads** → Use **whole-file cache mode**.

```python
env_var = {
    "DATASET_MOUNT_BLOCK_BASED_CACHE_ENABLED": True,
    "DATASET_MOUNT_MEMORY_CACHE_SIZE": 0
}
job = command(environment_variables=env_var)
```

> **Handling Many Small Files**

- **Combine small files** into ZIP or TAR archives.
- **Use Premium SSD storage** for low-latency reads.

---

## **9. Scaling & Storage Optimization**

> **Storage Account Limits**

- **Default egress limit**: 120 Gbit/s
- **High-speed VMs (A100, V100)** → Need **multiple storage accounts** if using 6+ nodes.

> **Scaling Across Storage Accounts**

- Use **multiple datastores** and distribute data across them.

```yaml
inputs:
  cifar_storage1:
    type: uri_folder
    path: azureml://datastores/storage1/paths/cifar
  cifar_storage2:
    type: uri_folder
    path: azureml://datastores/storage2/paths/cifar
```

---

## **10. Reading Azure ML v1 Data Assets in v2**

### **Convert v1 FileDataset**

```python
filedataset_asset = ml_client.data.get(name="my_filedataset", version="1")

inputs = {
    "input_data": Input(type=AssetTypes.MLTABLE, path=filedataset_asset.id, mode=InputOutputModes.EVAL_MOUNT)
}
```

### **Convert v1 TabularDataset**

```python
tabulardataset_asset = ml_client.data.get(name="my_table", version="1")

inputs = {
    "input_data": Input(type=AssetTypes.MLTABLE, path=tabulardataset_asset.id, mode=InputOutputModes.DIRECT)
}
```

---

## **Key Takeaways**

✅ **Use Datastore URIs** for seamless cloud storage access.  
✅ **Use `ro_mount` for large datasets** & **`download` for small datasets**.  
✅ **Optimize performance** by tuning parallel downloads and caching.  
✅ **Scale storage accounts** when using **6+ GPU nodes**.  
✅ **For small files,** use **archiving (ZIP, TAR)** to improve read speed.

By following these **best practices**, Azure ML jobs can **efficiently read, process, and write** large datasets with minimal latency and maximum scalability. 🚀
