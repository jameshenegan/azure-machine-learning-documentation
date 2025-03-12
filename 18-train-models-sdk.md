# Train models with Azure Machine Learning SDK

### \*\*Detailed Summary: Train Models with Azure Machine Learning SDK

Azure Machine Learning (Azure ML) provides multiple ways to submit ML training jobs, including:

1. **Azure CLI (`ml` extension)**
2. **Python SDK (`azure-ai-ml`)**
3. **REST API**

This article explains how to:

- **Train models** using different interfaces.
- **Submit jobs** to Azure ML.
- **Register trained models** for future use.

---

## **1. Prerequisites**

Before training models, ensure you have:

- An **Azure subscription**.
- An **Azure Machine Learning workspace**.
- The **Azure ML SDK for Python** (if using Python):
  ```bash
  pip install azure-ai-ml
  ```
- A cloned **Azure ML examples GitHub repository** (for sample code):
  ```bash
  git clone --depth 1 https://github.com/Azure/azureml-examples
  ```

---

## **2. Example: Training Job Using the Iris Dataset**

The examples use the **Iris flower dataset** to train an MLFlow model using **LightGBM**.

### **Training in the Cloud**

To train in the cloud, you need to:

1. **Connect to Azure ML workspace**.
2. **Create a compute cluster** (or use serverless compute).
3. **Submit a training job**.

---

## **3. Connect to Azure ML Workspace**

### **Using Python SDK**

```python
from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential

# Define Azure ML workspace details
subscription_id = '<SUBSCRIPTION_ID>'
resource_group = '<RESOURCE_GROUP>'
workspace_name = '<AZUREML_WORKSPACE_NAME>'

# Connect to workspace
ml_client = MLClient(DefaultAzureCredential(), subscription_id, resource_group, workspace_name)
```

---

## **4. Create a Compute Resource for Training**

Azure ML Compute is a managed cloud resource for ML workloads.

### **Using Python SDK**

```python
from azure.ai.ml.entities import AmlCompute

# Define compute cluster name
cpu_compute_target = "cpu-cluster"

try:
    ml_client.compute.get(cpu_compute_target)
except Exception:
    print("Creating a new compute cluster...")
    compute = AmlCompute(name=cpu_compute_target, size="STANDARD_D2_V2", min_instances=0, max_instances=4)
    ml_client.compute.begin_create_or_update(compute).result()
```

> **Note:** To use **serverless compute**, **skip this step**.

---

## **5. Submit the Training Job**

Azure ML allows job submission using the **Python SDK, CLI, and REST API**.

### **Using Python SDK**

```python
from azure.ai.ml import command, Input

# Define job command
command_job = command(
    code="./src",
    command="python main.py --iris-csv ${{inputs.iris_csv}} --learning-rate ${{inputs.learning_rate}} --boosting ${{inputs.boosting}}",
    environment="AzureML-lightgbm-3.2-ubuntu18.04-py37-cpu@latest",
    inputs={
        "iris_csv": Input(type="uri_file", path="https://azuremlexamples.blob.core.windows.net/datasets/iris.csv"),
        "learning_rate": 0.9,
        "boosting": "gbdt",
    },
    compute="cpu-cluster",
)

# Submit job
returned_job = ml_client.jobs.create_or_update(command_job)
print("Job submitted:", returned_job.studio_url)
```

### **Job Parameters**

| **Parameter**     | **Description**                                           |
| ----------------- | --------------------------------------------------------- |
| **`code`**        | Path to training script.                                  |
| **`command`**     | The execution command for the script.                     |
| **`environment`** | Specifies the Azure ML environment (pre-built or custom). |
| **`inputs`**      | Defines input dataset paths and parameters.               |
| **`compute`**     | The compute target for training.                          |

> **Tip:** For **serverless compute**, remove `"compute":"cpu-cluster"`.

---

## **6. Monitor the Job**

After submission, monitor the job in **Azure ML Studio** using the returned **job URL**:

```python
print(returned_job.studio_url)
```

Alternatively, check status using:

```python
print(returned_job.status)
```

---

## **7. Register the Trained Model**

Once the training job completes, register the model in the **Azure ML workspace**.

### **Using Python SDK**

```python
from azure.ai.ml.entities import Model
from azure.ai.ml.constants import AssetTypes

# Register trained model
run_model = Model(
    path="azureml://jobs/{}/outputs/artifacts/paths/model/".format(returned_job.name),
    name="iris-classifier",
    description="Trained model for Iris dataset",
    type=AssetTypes.MLFLOW_MODEL,
)

ml_client.models.create_or_update(run_model)
```

> **Tip:** The `returned_job.name` is used as part of the model path.

---

## **8. Train Models Using Azure CLI**

Azure ML CLI provides a command-line interface for job submission.

### **Submit a Training Job**

```bash
az ml job create --file job.yml
```

> **Use Case:** CLI is ideal for **scripting and automation**.

---

## **9. Train Models Using REST API**

Azure ML's REST API allows job submission via HTTP requests.

### **Submit a Training Job via REST API**

```json
POST https://<workspace-region>.api.azureml.ms/mlflow/v2.0/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.MachineLearningServices/workspaces/<workspace-name>/jobs
Content-Type: application/json
Authorization: Bearer <access-token>

{
  "name": "iris-training-job",
  "properties": {
    "computeId": "cpu-cluster",
    "command": "python main.py --iris-csv https://azuremlexamples.blob.core.windows.net/datasets/iris.csv"
  }
}
```

> **Use Case:** REST API is useful for **custom integrations** and **enterprise applications**.

---

## **10. Next Steps**

After training, the next step is **model deployment**:

- **Deploy models using Azure ML Endpoints**.
- **Deploy models as REST API services**.
- **Deploy models in AKS, Azure Functions, or IoT Edge**.

---

## **Key Takeaways**

✅ **Multiple training options**: Python SDK, CLI, and REST API.  
✅ **Train models in the cloud** using managed **compute clusters**.  
✅ **Track and monitor jobs** using Azure ML Studio.  
✅ **Register trained models** for deployment and reuse.  
✅ **Automate workflows** with CLI and REST API.

By leveraging **Azure ML**, you can **efficiently train, manage, and deploy models** at scale. 🚀
