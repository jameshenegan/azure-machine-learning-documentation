# Access data from Azure cloud storage during interactive development

https://learn.microsoft.com/en-us/azure/machine-learning/how-to-access-data-interactive

### **Detailed Summary: Access Data from Azure Cloud Storage During Interactive Development**

Azure Machine Learning (Azure ML) supports seamless **interactive data access** for exploratory data analysis (EDA), data preprocessing, and ML model prototyping. This article covers methods to **access and process data stored in Azure cloud storage** during interactive development using **Azure ML datastores, fsspec, mltable, and azcopy**.

---

## **1. Overview**

Interactive development is typically done in **Jupyter notebooks or Python consoles**, requiring easy and efficient access to cloud-stored data. This article explains:

- **Using Datastore URIs as filesystems.**
- **Reading data into Pandas using `mltable`.**
- **Downloading data explicitly using `azcopy`.**

> **Prerequisites:**

- An **Azure ML workspace**.
- An **Azure ML datastore**.
- Installed Python packages:
  ```bash
  pip install -U azureml-fsspec==1.3.1 mltable azure-ai-ml
  ```

> **Best Practice:** Use an **Azure ML Compute Instance** for seamless cloud storage access.

---

## **2. Access Data Using Datastore URIs (Filesystem-like)**

An **Azure ML datastore** acts as a **reference to an Azure storage account**, simplifying credential management and providing a **unified API for accessing different storage types (Blob, ADLS, File Share).**

### **Benefits of Using Datastore URIs**

✅ **Unified API:** Interacts with different storage types.  
✅ **Secure Access:** Supports both **identity-based (Microsoft Entra ID)** and **credential-based authentication (SAS tokens, service principal).**  
✅ **Direct Access:** Allows **copy-pasting** datastore URIs from the **Azure ML Studio UI**.

### **Datastore URI Format**

```python
uri = "azureml://subscriptions/<sub_id>/resourcegroups/<rg_name>/workspaces/<workspace>/datastores/<datastore>/paths/<path_to_file>"
```

### **Example: Read CSV File into Pandas**

```python
import pandas as pd

df = pd.read_csv("azureml://subscriptions/<sub_id>/resourcegroups/<rg_name>/workspaces/<workspace>/datastores/<datastore>/paths/data/sample.csv")
df.head()
```

### **Example: List Files Using fsspec**

The **fsspec (filesystem spec) API** provides filesystem-like interactions with cloud storage.

```python
from azureml.fsspec import AzureMachineLearningFileSystem

fs = AzureMachineLearningFileSystem("azureml://subscriptions/<sub_id>/resourcegroups/<rg_name>/workspaces/<workspace>/datastores/<datastore>")

# List files
fs.ls()

# Open a file
with fs.open("folder1/sample.csv") as f:
    data = f.read()
```

---

## **3. Upload & Download Files Using AzureMachineLearningFileSystem**

### **Upload Files**

```python
fs.upload(lpath="local_folder/file.csv", rpath="data/fsspec", recursive=False, overwrite="MERGE_WITH_OVERWRITE")
```

### **Download Files**

```python
fs.download(rpath="data/fsspec/sample.csv", lpath="local_folder/", recursive=False)
```

### **Overwrite Modes**

| **Mode**                | **Description**                       |
| ----------------------- | ------------------------------------- |
| `APPEND`                | Keeps the original file if it exists. |
| `FAIL_ON_FILE_CONFLICT` | Throws an error if the file exists.   |
| `MERGE_WITH_OVERWRITE`  | Overwrites the existing file.         |

---

## **4. Read & Process Data in Pandas Using `mltable`**

### **Why Use `mltable`?**

✅ **Handles multiple formats** (CSV, Parquet, Delta, JSON).  
✅ **Supports reading folders & glob patterns**.  
✅ **Works with both cloud and local data sources.**

### **Read a CSV File**

```python
import mltable

path = {'file': 'abfss://<filesystem>@<account>.dfs.core.windows.net/<folder>/<file>.csv'}
tbl = mltable.from_delimited_files(paths=[path])
df = tbl.to_pandas_dataframe()
df.head()
```

### **Read Multiple CSVs in a Folder**

```python
fs = AzureMachineLearningFileSystem("azureml://subscriptions/<sub_id>/resourcegroups/<rg_name>/workspaces/<workspace>/datastores/<datastore>")
dflist = [pd.read_csv(fs.open(file)) for file in fs.glob("/folder/*.csv")]
df = pd.concat(dflist)
df.head()
```

### **Read Parquet Files**

```python
path = {'pattern': 'abfss://<filesystem>@<account>.dfs.core.windows.net/folder/*.parquet'}
tbl = mltable.from_parquet_files(paths=[path])
df = tbl.to_pandas_dataframe()
df.head()
```

---

## **5. Read Large Data Efficiently**

> **⚠️ Pandas cannot handle very large datasets efficiently.**  
> ✅ **Use `dask` for large-scale data processing.**

### **Read Large CSV into Dask**

```python
import dask.dataframe as dd

df = dd.read_csv("azureml://subscriptions/<sub_id>/resourcegroups/<rg_name>/workspaces/<workspace>/datastores/<datastore>/paths/folder/*.csv")
df.head()
```

### **Random Sampling from Large Data**

```python
tbl = tbl.take_random_sample(probability=0.3)
df = tbl.to_pandas_dataframe()
df.head()
```

---

## **6. Read Data from Azure Databricks Filesystem (DBFS)**

### **Authenticate to DBFS**

```bash
export ADB_PAT=<databricks_personal_access_token>
```

### **Read CSV File from DBFS**

```python
import pandas as pd

storage_options = {
    'instance': 'adb-12345678.azuredatabricks.net',
    'token': os.getenv("ADB_PAT")
}

df = pd.read_csv('dbfs:/mnt/data/sample.csv', storage_options=storage_options)
df.head()
```

---

## **7. Read & Process Images in PyTorch**

### **Read an Image with `Pillow`**

```python
from PIL import Image

with fs.open('/folder/image.jpeg') as f:
    img = Image.open(f)
    img.show()
```

### **PyTorch Dataset for Image Processing**

```python
import os
import pandas as pd
from PIL import Image
from torch.utils.data import Dataset

class CustomImageDataset(Dataset):
    def __init__(self, filesystem, annotations_file, img_dir):
        self.fs = filesystem
        self.img_labels = pd.read_csv(self.fs.open(annotations_file))
        self.img_dir = img_dir

    def __len__(self):
        return len(self.img_labels)

    def __getitem__(self, idx):
        img_path = os.path.join(self.img_dir, self.img_labels.iloc[idx, 0])
        image = Image.open(self.fs.open(img_path))
        label = self.img_labels.iloc[idx, 1]
        return image, label
```

---

## **8. Explicitly Download Data Using `azcopy`**

For **large datasets**, **download files to local SSD storage** using `azcopy` for **better performance**.

### **Steps to Download Data Using `azcopy`**

```bash
# Create local directory
mkdir /home/azureuser/data

# Login to azcopy
azcopy login

# Copy data
azcopy cp "https://<storage_account>.blob.core.windows.net/container/path/" "/home/azureuser/data" --recursive
```

> **⚠️ Avoid downloading data into `/home/azureuser/cloudfiles/code/`**  
> Instead, use **`/home/azureuser/`** for better performance.

---

## **9. Key Takeaways**

✅ **Use Datastore URIs** to seamlessly access Azure Storage like a filesystem.  
✅ **Use `fsspec` to list, open, and manipulate files in Azure storage.**  
✅ **Use `mltable` to read structured data efficiently into Pandas or Dask.**  
✅ **Use `azcopy` for explicit downloads to local storage for large datasets.**  
✅ **For large datasets, use `dask` or Spark-based solutions instead of Pandas.**  
✅ **For deep learning, integrate Azure ML datastores with PyTorch datasets.**

By leveraging these **best practices**, you can efficiently access, process, and analyze cloud-stored data during interactive ML development. 🚀
