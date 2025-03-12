# Share models, components, and environments across workspaces with registries

https://learn.microsoft.com/en-us/azure/machine-learning/how-to-share-models-pipelines-across-workspaces-with-registries

### **Detailed Summary: Share Models, Components, and Environments Across Workspaces with Azure Machine Learning Registries**

---

## **Overview**

Azure Machine Learning **registries** enable collaboration across workspaces by allowing users to **share models, components, and environments** within an organization. This ensures **cross-workspace MLOps** and enhances productivity by **reusing and standardizing assets** across teams.

### **Key Use Cases:**

1. **Cross-workspace MLOps** – Train a model in a development (dev) workspace and deploy it to test and production (prod) workspaces with full lineage tracking.
2. **Sharing and Reuse** – Publish models, components, and environments to a **central registry** so teams can **reuse assets** instead of recreating them.

---

## **1. Prerequisites**

Before using Azure ML registries, ensure you have:

- **An Azure subscription** (free or paid).
- **An Azure Machine Learning registry** to share assets.
- **An Azure Machine Learning workspace** (must be in a supported region).
- **Azure CLI with the ML extension installed** or the **Python SDK v2**.

### **Set Up Defaults for Azure CLI**

```bash
az account set --subscription <subscription-id>
az configure --defaults workspace=<workspace-name> group=<resource-group> location=<location>
```

To check current defaults:

```bash
az configure -l
```

### **Clone the Example Repository**

```bash
git clone https://github.com/Azure/azureml-examples
cd azureml-examples
```

---

## **2. Creating and Sharing Assets Using a Registry**

### **2.1 Create an Environment in the Registry**

Environments define **Docker containers and Python dependencies** for training and deployment.

#### **Define the Environment (YAML)**

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/environment.schema.json
name: SKLearnEnv
version: 1
build:
  path: ./env_train
```

#### **Create the Environment in the Registry**

```bash
az ml environment create --file env_train.yml --registry-name <registry-name>
```

#### **List or Show Environments**

```bash
az ml environment list --registry-name <registry-name>
az ml environment show --name SKLearnEnv --version 1 --registry-name <registry-name>
```

---

### **2.2 Create a Component in the Registry**

A **component** packages the **code, environment, inputs, and outputs** of a pipeline step, making it reusable across workspaces.

#### **Define the Component (YAML)**

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/commandComponent.schema.json
name: train_linear_regression_model
display_name: TrainLinearRegressionModel
version: 1
type: command
inputs:
  training_data:
    type: uri_folder
  test_split_ratio:
    type: number
    min: 0
    max: 1
    default: 0.2
outputs:
  model_output:
    type: mlflow_model
  test_data:
    type: uri_folder
code: ./train_src
environment: azureml://registries/<registry-name>/environments/SKLearnEnv/versions/1
command: >-
  python train.py 
  --training_data ${{inputs.training_data}} 
  --test_data ${{outputs.test_data}} 
  --model_output ${{outputs.model_output}}
  --test_split_ratio ${{inputs.test_split_ratio}}
```

#### **Create the Component in the Registry**

```bash
az ml component create --file train.yml --registry-name <registry-name>
```

#### **List or Show Components**

```bash
az ml component list --registry-name <registry-name>
az ml component show --name train_linear_regression_model --version 1 --registry-name <registry-name>
```

---

## **3. Running a Pipeline Using a Component from the Registry**

A **pipeline job** can use components stored in a registry while still operating within a **local workspace**.

#### **Modify Pipeline YAML to Use the Registry Component**

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/pipelineJob.schema.json
type: pipeline
display_name: nyc_taxi_data_regression_single_job
description: Single job pipeline to train regression model

jobs:
  train_job:
    type: command
    component: azureml://registries/<registry-name>/components/train_linear_regression_model/versions/1
    compute: azureml:cpu-cluster
    inputs:
      training_data:
        type: uri_folder
        path: ./data_transformed
    outputs:
      model_output:
        type: mlflow_model
```

#### **Run the Pipeline in the Workspace**

```bash
az ml job create --file single-job-pipeline.yml
```

#### **Run in Different Workspaces**

```bash
az ml job create --file single-job-pipeline.yml --workspace-name test-workspace --resource-group <resource-group>
az ml job create --file single-job-pipeline.yml --workspace-name prod-workspace --resource-group <resource-group>
```

---

## **4. Registering a Model in the Registry**

After training, **register the model** so it can be **shared across workspaces**.

### **Option 1: Register from Local Files**

```bash
az ml model create --name nyc-taxi-model --version 1 --type mlflow_model --path ./artifacts/model/ --registry-name <registry-name>
```

### **Option 2: Share a Model from a Workspace**

```bash
az ml model create --name nyc-taxi-model --version 1 --type mlflow_model --path azureml://jobs/<train-job-name>/outputs/artifacts/paths/model
az ml model share --name nyc-taxi-model --version 1 --registry-name <registry-name>
```

#### **List or Show Models**

```bash
az ml model list --registry-name <registry-name>
az ml model show --name nyc-taxi-model --version 1 --registry-name <registry-name>
```

---

## **5. Deploy a Model from the Registry to an Online Endpoint**

Models in the registry can be deployed to an **online endpoint** for **real-time inference**.

### **Step 1: Create an Online Endpoint**

```bash
az ml online-endpoint create --name reg-ep-1234
```

### **Step 2: Define the Deployment (YAML)**

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/managedOnlineDeployment.schema.json
name: demo
endpoint_name: reg-ep-1234
model: azureml://registries/<registry-name>/models/nyc-taxi-model/versions/1
instance_type: Standard_DS2_v2
instance_count: 1
```

### **Step 3: Deploy the Model**

```bash
az ml online-deployment create --file deploy.yml --all-traffic
```

### **Step 4: Submit a Sample Inference Request**

```bash
ENDPOINT_KEY=$(az ml online-endpoint get-credentials -n reg-ep-1234 -o tsv --query primaryKey)
SCORING_URI=$(az ml online-endpoint show -n reg-ep-1234 -o tsv --query scoring_uri)

curl --request POST "$SCORING_URI" --header "Authorization: Bearer $ENDPOINT_KEY" --header 'Content-Type: application/json' --data @./scoring-data.json
```

---

## **6. Clean Up Resources**

If the deployment is no longer needed:

```bash
az ml online-endpoint delete --name reg-ep-1234 --yes --no-wait
```

---

## **Summary & Key Takeaways**

✔ **Azure ML registries allow cross-workspace collaboration** by enabling sharing of **models, components, and environments**.  
✔ **Training jobs can use registry components** while running in any workspace.  
✔ **Registered models can be deployed from the registry** to any online endpoint in an Azure ML workspace.  
✔ **Standardized environments ensure consistency** across different machine learning workflows.  
✔ **Automation via Azure CLI and Python SDK simplifies registry management**.

---

## **Next Steps**

📌 **Learn More About Azure ML Registries**:

- [Create and Manage Registries](https://docs.microsoft.com/azure/machine-learning/how-to-manage-registries)
- [Deploy ML Models](https://docs.microsoft.com/azure/machine-learning/how-to-deploy-models)
- [Manage Model Versions](https://docs.microsoft.com/azure/machine-learning/how-to-manage-models)

🚀 **Start Using Azure ML Registries for Scalable Machine Learning Pipelines!**
