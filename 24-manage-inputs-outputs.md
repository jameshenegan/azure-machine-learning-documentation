# Manage inputs and outputs for components and pipelines

https://learn.microsoft.com/en-us/azure/machine-learning/how-to-manage-inputs-outputs-pipeline

### **Detailed Summary: Manage Inputs and Outputs for Components and Pipelines in Azure Machine Learning**

## **Overview**

Azure Machine Learning (ML) pipelines support **inputs and outputs** at both the **component** and **pipeline** levels. Inputs and outputs enable **data flow between components** and allow users to **modify pipeline parameters dynamically**. This guide covers how to manage different types of inputs and outputs, define data flow, and handle serialization.

---

## **1. Inputs and Outputs in Azure ML Pipelines**

- **Component Level:** Defines inputs/outputs for each step of the pipeline.
- **Pipeline Level:** Passes data between steps and allows parameter tuning.

🔹 **Key Benefits:**

- Enables **modular design**.
- Supports **dynamic parameterization**.
- Allows **data/model sharing** between components.

### **Input and Output Types**

| **Type**            | **Inputs** | **Outputs** | Description              |
| ------------------- | ---------- | ----------- | ------------------------ |
| **Data Types**      | ✅         | ✅          | Standard data formats    |
| `uri_file`          | ✅         | ✅          | Path to a single file    |
| `uri_folder`        | ✅         | ✅          | Path to a folder         |
| `mltable`           | ✅         | ✅          | MLTable dataset          |
| **Model Types**     | ✅         | ✅          | Model serialization      |
| `mlflow_model`      | ✅         | ✅          | MLflow-trained model     |
| `custom_model`      | ✅         | ✅          | Custom model format      |
| **Primitive Types** | ✅         | ❌          | Direct parameter passing |
| `string`            | ✅         | ❌          | Text input               |
| `number`            | ✅         | ❌          | Numeric input            |
| `integer`           | ✅         | ❌          | Whole number input       |
| `boolean`           | ✅         | ❌          | True/False input         |

⚠ **Primitive type outputs are NOT supported.**

---

## **2. Data Flow in Pipelines**

### **Example: NYC Taxi Data Regression Pipeline**

- **Train Component:** Uses a `number` input called `test_split_ratio`.
- **Prep Component:** Uses a `uri_folder` output where processed CSV files are stored.
- **Train Component:** Saves a trained model as an `mlflow_model`.

---

## **3. Output Serialization**

- Outputs are stored in files **within an Azure Storage account**.
- Components must **explicitly serialize outputs** (e.g., save a Pandas DataFrame as a CSV).
- **No predefined serialization method**—users can choose how to save and read files.

---

## **4. Input and Output Paths**

### **Supported Locations**

| **Location**            | **Input**          | **Output** | **Example Path**                                       |
| ----------------------- | ------------------ | ---------- | ------------------------------------------------------ |
| **Local Filesystem**    | ✅                 | ❌         | `./home/user/data/my_data`                             |
| **Public URL**          | ✅                 | ❌         | `https://raw.githubusercontent.com/.../titanic.csv`    |
| **Azure Storage**       | ⚠️ Not Recommended | ✅         | `wasbs://container@account.blob.core.windows.net/path` |
| **Azure ML Datastore**  | ✅                 | ✅         | `azureml://datastores/my_datastore/paths/my_data`      |
| **Azure ML Data Asset** | ✅                 | ✅         | `azureml:my_data:<version>`                            |

⚠ **Using Azure Storage directly for inputs is discouraged** due to authentication requirements.

---

## **5. Data Access Modes**

| **Type**            | **Upload** | **Download** | **Read-Only Mount (`ro_mount`)** | **Read-Write Mount (`rw_mount`)** | **Direct Access (`direct`)** |
| ------------------- | ---------- | ------------ | -------------------------------- | --------------------------------- | ---------------------------- |
| `uri_folder` Input  | ❌         | ✅           | ✅                               | ❌                                | ✅                           |
| `uri_file` Input    | ❌         | ✅           | ✅                               | ❌                                | ✅                           |
| `mltable` Input     | ❌         | ✅           | ✅                               | ✅                                | ✅                           |
| `uri_folder` Output | ✅         | ❌           | ✅                               | ❌                                | ❌                           |
| `uri_file` Output   | ✅         | ❌           | ✅                               | ❌                                | ❌                           |
| `mltable` Output    | ✅         | ❌           | ✅                               | ✅                                | ❌                           |

✅ **Recommended:** Use `ro_mount` or `rw_mount` for most cases.

---

## **6. Promoting Component Inputs/Outputs to Pipeline Level**

### **Why?**

- Allows modifying inputs dynamically when submitting pipeline jobs.
- Useful for **REST API-triggered pipelines**.

### **Example Pipeline YAML with Promoted Inputs/Outputs**

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/pipelineJob.schema.json
type: pipeline
display_name: registered_components_pipeline

inputs:
  pipeline_max_epochs: 20
  pipeline_learning_rate: 1.8
  pipeline_schedule: "time-based"

outputs:
  pipeline_trained_model:
    mode: upload
  pipeline_scored_data:
    mode: upload
  pipeline_evaluation_report:
    mode: upload

jobs:
  train_job:
    type: command
    component: azureml:train_model@latest
    inputs:
      max_epochs: ${{parent.inputs.pipeline_max_epochs}}
      learning_rate: ${{parent.inputs.pipeline_learning_rate}}
    outputs:
      model_output: ${{parent.outputs.pipeline_trained_model}}
```

🔹 **Key Highlights**

- **Pipeline Inputs/Outputs are defined at the root level**.
- **Component Jobs reference these values dynamically** (`${{parent.inputs.pipeline_max_epochs}}`).

---

## **7. Optional Inputs**

By default, **inputs are required**. To make an input **optional**, set:

```yaml
optional: true
```

### **Example**

```yaml
inputs:
  max_epochs:
    type: integer
    optional: true
  learning_rate:
    type: number
    default: 0.01
    optional: true
```

⚠ **Optional outputs are NOT supported.**

---

## **8. Customizing Output Paths**

By default, outputs are stored in the workspace's **default datastore**:

```yaml
azureml://datastores/${{default_datastore}}/paths/${{name}}/${{output_name}}
```

### **Custom Output Example**

```bash
output_path="azureml://datastores/my_datastore/paths/my_output_data"
az ml job create -f pipeline.yml --set outputs.pipeline_trained_model.path=$output_path
```

---

## **9. Downloading Pipeline Outputs**

**Pipeline-Level Downloads**

```bash
az ml job download --all -n <JOB_NAME> -g <RESOURCE_GROUP_NAME> -w <WORKSPACE_NAME> --subscription <SUBSCRIPTION_ID>
```

**Component-Level Downloads**

```bash
az ml job list --parent-job-name <JOB_NAME> -g <RESOURCE_GROUP_NAME> -w <WORKSPACE_NAME>
az ml job download --all -n <CHILD_JOB_NAME>
```

---

## **10. Registering Outputs as Named Assets**

Outputs can be registered as **named assets** for reuse.

### **Register Pipeline Output**

```yaml
outputs:
  pipeline_trained_model:
    type: mltable
    name: trained_model_asset
    version: "1"
```

### **Register Component Output**

```yaml
outputs:
  component_output:
    type: uri_folder
    name: component_output_asset
    version: "1"
```

---

## **🚀 Summary**

✅ **Azure ML supports various input/output types** (data, models, parameters).  
✅ **Data serialization is required for outputs** before storage.  
✅ **Pipeline inputs/outputs allow dynamic parameterization** and **REST API integration**.  
✅ **Multiple data access modes** (`mount`, `direct`, `upload/download`).  
✅ **Components can promote inputs/outputs to pipeline level** for flexibility.  
✅ **Outputs can be registered as named assets** for future reuse.

By **properly managing inputs and outputs**, you can build **scalable, modular, and efficient** ML workflows in Azure ML. 🚀
