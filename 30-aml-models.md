# Work with registered models in Azure Machine Learning

https://learn.microsoft.com/en-us/azure/machine-learning/how-to-manage-models

### **Detailed Summary: Working with Registered Models in Azure Machine Learning**

---

## **Overview**

Azure Machine Learning (Azure ML) enables users to **register, version, manage, and deploy machine learning models**. Registered models are stored in a **centralized model registry** within an Azure ML workspace, making them easy to **track, share, and deploy**.

This article covers:

- **Registering models** using **Azure ML Studio UI, Azure CLI, and Python SDK**.
- **Different storage locations and access modes** for models.
- **Using models as inputs or outputs in training jobs**.
- **Managing the model lifecycle** (listing, updating, archiving, and deploying).

---

## **1. Model Registration in Azure ML**

**Model registration** allows you to store and version models in Azure ML. It supports various **storage locations**:

| **Storage Location**              | **Path Syntax**                                                               |
| --------------------------------- | ----------------------------------------------------------------------------- |
| **Local computer**                | `<model-folder>/<model-filename>`                                             |
| **Azure ML Datastore**            | `azureml://datastores/<datastore-name>/paths/<path_on_datastore>`             |
| **Azure ML Job Output**           | `azureml://jobs/<job-name>/outputs/<output-name>/paths/<path-to-model>`       |
| **MLflow Job**                    | `runs:/<run-id>/<path-to-model>`                                              |
| **Registered Model in Workspace** | `azureml:<model-name>:<version>`                                              |
| **Registered Model in Registry**  | `azureml://registries/<registry-name>/models/<model-name>/versions/<version>` |

---

## **2. Supported Model Types & Access Modes**

Azure ML supports **multiple model types**, including:

- **Custom models** (non-MLflow standard formats).
- **MLflow models** (stored with MLmodel files).
- **Triton models** (used for high-performance inference).

### **Model Access Modes**

Models can be accessed in different ways depending on the use case:

| **Type**                 | **Upload** | **Download** | **Read-Only Mount (ro_mount)** | **Read-Write Mount (rw_mount)** | **Direct (URI Reference)** |
| ------------------------ | ---------- | ------------ | ------------------------------ | ------------------------------- | -------------------------- |
| **Custom File Input**    | ✗          | ✓            | ✓                              | ✗                               | ✓                          |
| **Custom Folder Input**  | ✗          | ✓            | ✓                              | ✗                               | ✓                          |
| **MLflow Input**         | ✗          | ✓            | ✓                              | ✗                               | ✗                          |
| **Custom File Output**   | ✓          | ✗            | ✗                              | ✓                               | ✓                          |
| **Custom Folder Output** | ✓          | ✗            | ✗                              | ✓                               | ✓                          |
| **MLflow Output**        | ✓          | ✗            | ✗                              | ✓                               | ✓                          |

---

## **3. Prerequisites**

Before registering and working with models in Azure ML, ensure you have:

- An **Azure ML workspace**.
- **Azure CLI** (v2.38.0 or later) installed.
- **Azure ML extension for CLI**:
  ```bash
  az extension add -n ml
  ```
- **Python SDK v2 installed**.

---

## **4. Register a Model in Azure ML**

Models can be registered using **Azure ML Studio UI, CLI, or Python SDK**.

### **4.1 Register via Studio UI**

1. Navigate to **Models** in the **Azure ML Studio**.
2. Click **Register Model** → Choose model source:
   - From **local files**.
   - From an **Azure ML job output**.
   - From an **Azure ML datastore**.
3. Specify **model type** (MLflow, Triton, or Custom).
4. Set **model name and optional metadata**.
5. Click **Register**.

### **4.2 Register via Azure CLI**

#### **Register a Model from a Local File**

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/model.schema.json
name: my-local-model
path: models/model.pkl
description: Model created from a local file.
```

```bash
az ml model create -f my-local-model.yml
```

#### **Register a Model from a Datastore**

```bash
az ml model create --name my-model --version 1 --path azureml://datastores/myblobstore/paths/models/cifar10/cifar.pt
```

#### **Register a Model from a Job Output**

- **MLflow Runs Format**:
  ```bash
  az ml model create --name my-registered-model --version 1 --path runs:/my_run_123/model/ --type mlflow_model
  ```
- **Azure ML Job Output Format**:
  ```bash
  az ml model create --name run-model-example --version 1 --path azureml://jobs/my_run_123/outputs/artifacts/paths/model/
  ```

### **4.3 Register via Python SDK**

```python
from azure.ai.ml import MLClient
from azure.ai.ml.entities import Model
from azure.identity import DefaultAzureCredential

ml_client = MLClient.from_config(DefaultAzureCredential())

model = Model(
    path="models/model.pkl",
    name="my_local_model",
    description="Registered from local file.",
    type="custom_model"
)

ml_client.models.create_or_update(model)
```

---

## **5. Using Registered Models in Training Jobs**

Registered models can be used as **inputs or outputs** in training jobs.

### **5.1 Using a Model as Input**

Define the model in a YAML job file:

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/commandJob.schema.json
command: |
  python train.py --model_path ${{inputs.my_model}}
inputs:
  my_model:
    type: mlflow_model
    path: azureml:my_model:latest
environment: azureml://registries/azureml/environments/sklearn-1.5/labels/latest
```

Run the job:

```bash
az ml job create -f train-job.yml
```

### **5.2 Writing a Model as Output**

```yaml
outputs:
  trained_model:
    type: mlflow_model
    path: azureml://datastores/myblobstore/paths/trained_model/
```

Run the job:

```bash
az ml job create --file train-output-job.yml
```

---

## **6. Managing Registered Models**

### **6.1 List Registered Models**

```bash
az ml model list
```

```python
for model in ml_client.models.list():
    print(model.name, model.version)
```

### **6.2 Get Model Details**

```bash
az ml model show --name my_model --version 1
```

### **6.3 Update Model Metadata**

```bash
az ml model update --name my_model --version 1 --set description="Updated description" --set tags.stage="Production"
```

### **6.4 Archive a Model**

```bash
az ml model archive --name my_model --version 1
```

---

## **7. Deploying Registered Models**

Registered models can be deployed to **batch or real-time endpoints**.

### **7.1 Deploy Model as an Online Endpoint**

```bash
az ml online-endpoint create --name my-endpoint --model azureml:my_model:latest
```

### **7.2 Deploy Model as a Batch Endpoint**

```bash
az ml batch-endpoint create --name batch-inference --model azureml:my_model:latest
```

---

## **8. Key Takeaways**

✔ **Azure ML Model Registry** stores, versions, and manages ML models.  
✔ Models can be registered from **local files, Azure ML jobs, or datastores**.  
✔ Models can be used as **inputs or outputs** in training jobs.  
✔ **CLI and Python SDK** allow full lifecycle management of registered models.  
✔ Registered models can be **deployed to batch or real-time endpoints**.

---

## **9. Next Steps**

- Learn how to **[Deploy MLflow models to online endpoints](https://learn.microsoft.com/azure/machine-learning/how-to-deploy-mlflow)**.
- Explore **[Model registry best practices](https://learn.microsoft.com/azure/machine-learning/how-to-manage-models)**.
- **Automate model deployment with CI/CD** using **Azure DevOps**.

🚀 **By leveraging Azure ML Model Registry, teams can efficiently manage and deploy models across different environments, ensuring reproducibility, tracking, and governance.**
