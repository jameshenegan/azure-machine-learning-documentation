# Tutorial: Upload, access, and explore your data in Azure Machine Learning

https://learn.microsoft.com/en-us/azure/machine-learning/tutorial-explore-data

### **Detailed Summary: Upload, Access, and Explore Data in Azure Machine Learning**

This **tutorial** explains how to **upload, access, and explore data** in **Azure Machine Learning (Azure ML)**. It covers **creating data assets, uploading data to cloud storage, accessing data from notebooks, and versioning datasets**. The goal is to facilitate **exploratory data analysis (EDA), preprocessing, and model prototyping**.

---

## **Key Steps in the Tutorial**

### **1. Prerequisites**

Before starting, ensure:

- You have an **Azure Machine Learning workspace**.
- You can **sign in to Azure ML Studio**.
- You have a **compute instance** (for running notebooks).

**Note:**  
If your workspace uses a **managed virtual network**, you might need to configure **outbound rules** to access public Python package repositories.

---

## **2. Set Up the Development Environment**

- **Open or create a notebook** in **Azure ML Studio**.
- **Set the kernel** to **Python 3.10 - SDK v2**.
- If needed, **authenticate** using Azure credentials.
- Open the notebook in **Visual Studio Code** for an enhanced development experience.

---

## **3. Download and Store Data in Azure ML**

### **Download Data to Compute Instance**

1. **Open a terminal in Azure ML Studio**.
2. **Navigate to your notebook directory**:
   ```bash
   cd get-started-notebooks  # Modify this path as needed
   ```
3. **Create a folder and download the dataset**:
   ```bash
   mkdir data
   cd data
   wget https://azuremlexamples.blob.core.windows.net/datasets/credit_card/default_of_credit_card_clients.csv
   ```
4. **Close the terminal** after downloading the data.

The dataset is a **credit card client dataset (CSV format)** from the **UC Irvine Machine Learning Repository**.

---

## **4. Create a Handle to the Workspace**

A **workspace handle** allows interaction with Azure ML resources.

### **Initialize ML Client**

```python
from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential

# Authenticate
credential = DefaultAzureCredential()

# Connect to workspace
ml_client = MLClient(
    credential=credential,
    subscription_id="<SUBSCRIPTION_ID>",
    resource_group_name="<RESOURCE_GROUP>",
    workspace_name="<AML_WORKSPACE_NAME>",
)
```

**Note:** The MLClient initialization **does not immediately connect** to the workspace—it connects **when first used**.

---

## **5. Upload Data to Cloud Storage**

Azure ML uses **Uniform Resource Identifiers (URIs)** to store data in **Azure Blob Storage, Data Lake, and public URLs**.

### **Create a Data Asset**

Instead of remembering long URIs, **data assets** store references to storage locations.

```python
from azure.ai.ml.entities import Data
from azure.ai.ml.constants import AssetTypes

# Path to local dataset
my_path = "./data/default_of_credit_card_clients.csv"

# Set version
v1 = "initial"

# Define Data Asset
my_data = Data(
    name="credit-card",
    version=v1,
    description="Credit card data",
    path=my_path,
    type=AssetTypes.URI_FILE,
)

# Create Data Asset
try:
    data_asset = ml_client.data.get(name="credit-card", version=v1)
    print(f"Data asset already exists. Name: {my_data.name}, version: {my_data.version}")
except:
    ml_client.data.create_or_update(my_data)
    print(f"Data asset created. Name: {my_data.name}, version: {my_data.version}")
```

### **Verify Data in Azure ML Studio**

- In **Azure ML Studio**, select **Data** from the left menu.
- Find the uploaded **credit-card** dataset in the **Data assets** tab.

---

## **6. Access Data in Notebooks**

Azure ML stores datasets in **datastores**, which provide:

- **Authentication** for secure access.
- **Storage options** (Azure Blob, Data Lake, etc.).
- **Easier access** without remembering URIs.

### **Read Data Using Pandas**

```python
import pandas as pd

df = pd.read_csv("azureml://subscriptions/<SUBSCRIPTION_ID>/resourcegroups/<RESOURCE_GROUP>/workspaces/<WORKSPACE_NAME>/datastores/<DATASTORE_NAME>/paths/data/default_of_credit_card_clients.csv")
df.head()
```

To avoid manually replacing placeholders, **use data assets** for easier access.

### **Install Required Library**

```python
%pip install -U azureml-fsspec
```

### **Load Data Using a Data Asset**

```python
# Retrieve Data Asset
data_asset = ml_client.data.get(name="credit-card", version=v1)
print(f"Data asset URI: {data_asset.path}")

# Load Data into Pandas
df = pd.read_csv(data_asset.path)
df.head()
```

---

## **7. Create a New Version of the Data Asset**

Before training, data **often needs cleaning**. The dataset has:

- **Duplicate headers**
- **An ID column (not useful for ML)**
- **A response variable with spaces in its name**

### **Clean and Save Data as a Parquet File**

```python
df = pd.read_csv(data_asset.path, header=1)
df.rename(columns={"default payment next month": "default"}, inplace=True)
df.drop("ID", axis=1, inplace=True)

# Save cleaned data in Parquet format
df.to_parquet("./data/cleaned-credit-card.parquet")
```

### **Upload Cleaned Data as a New Version**

Each data asset version **requires a unique identifier**.

```python
import time
from azure.ai.ml.entities import Data

# Generate a unique version name
v2 = "cleaned" + time.strftime("%Y.%m.%d.%H%M%S", time.gmtime())

# Define new Data Asset
my_data = Data(
    name="credit-card",
    version=v2,
    description="Cleaned credit card dataset",
    tags={"training_data": "true", "format": "parquet"},
    path="./data/cleaned-credit-card.parquet",
    type=AssetTypes.URI_FILE,
)

# Create Data Asset
my_data = ml_client.data.create_or_update(my_data)

print(f"Data asset created. Name: {my_data.name}, version: {my_data.version}")
```

---

## **8. Compare the Original and Cleaned Data**

```python
# Retrieve both versions
data_asset_v1 = ml_client.data.get(name="credit-card", version=v1)
data_asset_v2 = ml_client.data.get(name="credit-card", version=v2)

# Print original data
print(f"V1 Data asset URI: {data_asset_v1.path}")
v1df = pd.read_csv(data_asset_v1.path)
print(v1df.head(5))

# Print cleaned data
print("____________________________________________________________________________________\n")
print(f"V2 Data asset URI: {data_asset_v2.path}")
v2df = pd.read_parquet(data_asset_v2.path)
print(v2df.head(5))
```

---

## **9. Clean Up Resources**

### **Stop Compute Instance**

If not in use:

1. Open **Azure ML Studio**.
2. Navigate to **Compute > Compute Instances**.
3. Select the instance and click **Stop**.

### **Delete Azure Resources**

To avoid unnecessary costs:

1. Go to the **Azure Portal**.
2. Open **Resource Groups**.
3. Select the **resource group** you created.
4. Click **Delete**.

---

## **Next Steps**

- **Learn more about data assets**: [Create Data Assets](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-create-data-assets)
- **Explore Azure ML Datastores**: [Create Datastores](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-access-data)
- **Move on to model training**: [Develop a Training Script](https://learn.microsoft.com/en-us/azure/machine-learning/tutorial-train-model)

---

## **Conclusion**

This tutorial provides a **step-by-step guide** to handling data in **Azure Machine Learning**:
✅ **Download and store data** in Azure ML.  
✅ **Create and manage data assets** to simplify access.  
✅ **Perform light data cleaning** and versioning.  
✅ **Use data assets for ML workflows** in **notebooks and scripts**.

By using **data assets and datastores**, Azure ML provides **secure, scalable, and efficient** data management for machine learning workflows. 🚀
