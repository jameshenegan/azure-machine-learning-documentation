# Train models with Azure Machine Learning

https://learn.microsoft.com/en-us/azure/machine-learning/concept-train-machine-learning-model

### **Detailed Summary: Train Models with Azure Machine Learning**

Azure Machine Learning (Azure ML) provides multiple ways to train models, ranging from **code-first** solutions using the SDK to **low-code** and **no-code** solutions such as Automated Machine Learning (AutoML) and Azure ML Designer.

---

## **1. Training Methods in Azure ML**

Azure ML offers various training methods suited for different users, from experienced data scientists to those with minimal coding knowledge.

| **Training Method**                     | **Description**                                                                                      |
| --------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **Command (`command()`)**               | A traditional way to train models by defining a training script, environment, and compute resources. |
| **Automated Machine Learning (AutoML)** | Automatically selects algorithms, tunes hyperparameters, and trains models with minimal coding.      |
| **Machine Learning Pipelines**          | Define multi-step workflows for **scheduled training**, **data preparation**, and **retraining**.    |
| **Azure ML Designer**                   | Drag-and-drop UI for training models, ideal for beginners and quick prototyping.                     |
| **Azure CLI**                           | Used for **scripting**, **automation**, and **job scheduling**.                                      |
| **VS Code**                             | Provides an extension to manage and run training jobs interactively.                                 |

> **Compute Targets**: Each training method can run on different compute resources, including **local machines, cloud-based VM clusters (Azure Machine Learning Compute), HDInsight, and Kubernetes clusters**.

---

## **2. Training Models Using Python SDK**

The **Azure ML Python SDK** is the most flexible and widely used method for training models.

### **Installation and Configuration**

Before starting, install the required SDK:

```bash
pip install azure-ai-ml
```

Then, configure your environment:

```python
from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential

ml_client = MLClient(
    DefaultAzureCredential(),
    subscription_id="your-subscription-id",
    resource_group_name="your-resource-group",
    workspace_name="your-aml-workspace",
)
```

---

### **3. Submitting a Training Job Using `command()`**

The `command()` function is used to submit a training job, specifying:

- **Training script**
- **Compute target**
- **Environment**
- **Input/Output Data Paths**

#### **Example: Running a Training Job**

```python
from azure.ai.ml import command, Input
from azure.ai.ml.constants import AssetTypes

job = command(
    code="./src",  # Folder containing training script
    command="python train.py --data ${{inputs.training_data}}",
    inputs={
        "training_data": Input(type=AssetTypes.URI_FOLDER, path="azureml:my_dataset:1")
    },
    compute="gpu-cluster",
    environment="azureml:my-custom-environment:latest",
)

ml_client.jobs.create_or_update(job)
```

> ✅ **Best Practice:** You can switch from **local compute** to **cloud compute** by simply updating the `compute` parameter.

---

## **4. Automated Machine Learning (AutoML)**

AutoML automates model selection, feature engineering, and hyperparameter tuning, making it ideal for non-experts.

### **Key Features**

- **Automated model selection** → Tests multiple ML algorithms.
- **Hyperparameter tuning** → Optimizes model parameters.
- **Parallel execution** → Runs experiments concurrently.

### **Example: Running an AutoML Classification Experiment**

```python
from azure.ai.ml import automl

job = automl.classification(
    compute="cpu-cluster",
    experiment_name="classification-experiment",
    training_data=Input(type=AssetTypes.MLTABLE, path="azureml:my_dataset:1"),
    target_column_name="label",
    primary_metric="accuracy",
)

ml_client.jobs.create_or_update(job)
```

> **Alternative:** You can also configure and run **AutoML in Azure ML Studio**.

---

## **5. Machine Learning Pipelines**

ML Pipelines allow you to create **end-to-end workflows** that include:

- **Data ingestion & preprocessing**
- **Model training & hyperparameter tuning**
- **Model evaluation & deployment**

### **Example: Simple ML Pipeline**

```python
from azure.ai.ml import dsl, Input, Output

@dsl.pipeline(
    compute="cpu-cluster",
    description="Pipeline that trains a model",
)
def my_training_pipeline(input_data):
    train_step = command(
        command="python train.py --data ${{inputs.input_data}}",
        inputs={"input_data": input_data},
        compute="cpu-cluster",
    )
    return train_step

pipeline_job = my_training_pipeline(input_data=Input(path="azureml:my_dataset:1"))
ml_client.jobs.create_or_update(pipeline_job)
```

> **Use Case:** Pipelines are useful for **retraining models on new data**, **batch processing**, and **CI/CD automation**.

---

## **6. Training Lifecycle in Azure ML**

When submitting a job, the following steps occur:

1. **Project Packaging**

   - The **project folder** (training script, dependencies) is **zipped** and **uploaded to Azure**.
   - Use `.amlignore` to exclude unnecessary files.

2. **Compute Cluster Scaling**

   - Azure ML **starts the compute cluster** based on the job requirements.

3. **Environment Setup**

   - The **Docker image** is **built/downloaded** to the compute node.
   - A **conda environment** is created based on dependencies.

4. **Job Execution**

   - The **training script runs** with the provided dataset and parameters.

5. **Results & Logging**

   - **Model outputs, logs, and metrics** are stored in Azure ML.

6. **Compute Cluster Scaling Down**
   - If using **Azure ML Compute**, the cluster **scales down automatically** after the job.

---

## **7. Training with Azure ML Designer**

The **Azure ML Designer** is a **no-code** drag-and-drop interface for:

- **Building ML models without coding**
- **Connecting datasets, transformations, and algorithms visually**
- **Training & deploying models interactively**

> **Best For:** Users with little to no coding experience.

---

## **8. Using Azure CLI for Model Training**

Azure ML CLI provides **command-line automation** for training models.

### **Example: Submitting a Training Job via CLI**

```bash
az ml job create --file job.yml
```

> **Use Case:** Useful for **CI/CD automation** and **scripting model retraining**.

---

## **9. Training Models with VS Code**

- **Azure ML extension for VS Code** allows:
  - Managing **training jobs**.
  - Viewing **logs & model outputs**.
  - Submitting **experiments**.

> **Best For:** Users who prefer **local development with cloud execution**.

---

## **10. Best Practices for Efficient Model Training**

### ✅ **1. Use Compute Clusters Effectively**

- Choose **GPU clusters** for deep learning.
- Use **serverless compute** for small workloads.

### ✅ **2. Optimize Data Handling**

- Use **MLTable** for structured data.
- Store large datasets in **Blob Storage**.

### ✅ **3. Automate Retraining with Pipelines**

- Use pipelines to **retrain models** periodically.

### ✅ **4. Track and Log Experiments**

- Store **model artifacts** and **logs** in Azure ML.

### ✅ **5. Use Hyperparameter Tuning**

- AutoML or **Grid Search** for best model performance.

---

## **Key Takeaways**

✅ **Multiple training methods**: Python SDK, AutoML, Pipelines, CLI, Designer.  
✅ **Seamless cloud integration**: Easily switch from local to cloud compute.  
✅ **Optimize performance**: Use clusters, pipelines, and AutoML.  
✅ **Track and manage models**: Use Azure ML logging and versioning.  
✅ **Automate retraining**: Set up pipelines for scheduled model updates.

By following these **best practices**, you can efficiently train, optimize, and manage ML models in Azure Machine Learning. 🚀
