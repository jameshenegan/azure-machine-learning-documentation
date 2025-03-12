# Tutorial: Create production machine learning pipelines

https://learn.microsoft.com/en-us/azure/machine-learning/tutorial-pipeline-python-sdk

### **Detailed Summary: Create Production Machine Learning Pipelines in Azure ML**

This tutorial guides you through **building a production-ready machine learning pipeline** in **Azure Machine Learning (Azure ML)** using the **Python SDK v2**. The pipeline is designed to handle **data preparation, model training, and model registration** for a **credit default prediction** task.

---

## **Key Objectives**

By the end of this tutorial, you will be able to:
✅ **Set up an Azure ML pipeline** using Python SDK v2.  
✅ **Create reusable ML components** for data preparation and model training.  
✅ **Register trained models** in Azure ML workspace.  
✅ **Automate the pipeline execution** for efficiency and scalability.

---

## **1. Prerequisites**

Before you begin:

- **An Azure Machine Learning workspace** is required.
- **Complete the tutorial**: [Upload, access, and explore your data in Azure ML](https://learn.microsoft.com/en-us/azure/machine-learning/tutorial-data-access).
- Ensure you have **a compute instance running** in Azure ML Studio.
- Install the required **Python SDK v2** dependencies.

⚠ **If using a managed virtual network, configure outbound rules for access to Python package repositories.**

---

## **2. Set Up the Development Environment**

- **Open a Jupyter Notebook** in Azure ML Studio.
- Clone the sample notebook:
  ```
  tutorials/get-started-notebooks/pipeline.ipynb
  ```
- **Start a compute instance** and **set the kernel to Python 3.10 - SDK v2**.
- **Optionally, open in VS Code** for a richer development experience.

---

## **3. Set Up Pipeline Resources**

Before building the pipeline, define the following resources:

- **Data asset** for training.
- **Compute resource** where the job will run.
- **Software environment** to execute the pipeline.

### **Connect to Azure ML Workspace**

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

### **Verify Connection**

```python
ws = ml_client.workspaces.get("<AML_WORKSPACE_NAME>")
print(ws.location, ":", ws.resource_group)
```

---

## **4. Access the Registered Data Asset**

Retrieve the previously registered **credit card dataset**.

```python
credit_data = ml_client.data.get(name="credit-card", version="initial")
print(f"Data asset URI: {credit_data.path}")
```

---

## **5. Define the Pipeline Environment**

Each pipeline step requires an **execution environment**.

### **Create a Conda Environment**

```python
import os

dependencies_dir = "./dependencies"
os.makedirs(dependencies_dir, exist_ok=True)

%%writefile {dependencies_dir}/conda.yaml
name: model-env
channels:
  - conda-forge
dependencies:
  - python=3.8
  - numpy=1.21.2
  - pip=21.2.4
  - scikit-learn=0.24.2
  - scipy=1.7.1
  - pandas>=1.1,<1.2
  - pip:
    - inference-schema[numpy-support]==1.3.0
    - xlrd==2.0.1
    - mlflow==2.4.1
    - azureml-mlflow==1.51.0
```

### **Register the Environment in Azure ML**

```python
from azure.ai.ml.entities import Environment

custom_env_name = "aml-scikit-learn"

pipeline_job_env = Environment(
    name=custom_env_name,
    description="Custom environment for Credit Card Defaults pipeline",
    tags={"scikit-learn": "0.24.2"},
    conda_file=os.path.join(dependencies_dir, "conda.yaml"),
    image="mcr.microsoft.com/azureml/openmpi4.1.0-ubuntu20.04:latest",
    version="0.2.0",
)
pipeline_job_env = ml_client.environments.create_or_update(pipeline_job_env)

print(f"Environment {pipeline_job_env.name} registered with version {pipeline_job_env.version}")
```

---

## **6. Build the Training Pipeline**

Azure ML **pipelines** consist of multiple **reusable components**.

---

### **Step 1: Create the Data Preparation Component**

This component **splits the dataset** into **training and testing sets**.

#### **Create the Component Directory**

```python
data_prep_src_dir = "./components/data_prep"
os.makedirs(data_prep_src_dir, exist_ok=True)
```

#### **Write the Data Preparation Script (`data_prep.py`)**

```python
%%writefile {data_prep_src_dir}/data_prep.py
import os
import argparse
import pandas as pd
from sklearn.model_selection import train_test_split
import mlflow

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--data", type=str, help="path to input data")
    parser.add_argument("--test_train_ratio", type=float, default=0.25)
    parser.add_argument("--train_data", type=str, help="path to train data")
    parser.add_argument("--test_data", type=str, help="path to test data")
    args = parser.parse_args()

    mlflow.start_run()

    credit_df = pd.read_csv(args.data, header=1, index_col=0)
    train_df, test_df = train_test_split(credit_df, test_size=args.test_train_ratio)

    train_df.to_csv(os.path.join(args.train_data, "data.csv"), index=False)
    test_df.to_csv(os.path.join(args.test_data, "data.csv"), index=False)

    mlflow.end_run()

if __name__ == "__main__":
    main()
```

#### **Register the Data Preparation Component**

```python
from azure.ai.ml import command
from azure.ai.ml import Input, Output

data_prep_component = command(
    name="data_prep_credit_defaults",
    display_name="Data preparation for training",
    description="Splits input data into train and test sets",
    inputs={"data": Input(type="uri_folder"), "test_train_ratio": Input(type="number")},
    outputs={"train_data": Output(type="uri_folder"), "test_data": Output(type="uri_folder")},
    code=data_prep_src_dir,
    command="python data_prep.py --data ${{inputs.data}} --test_train_ratio ${{inputs.test_train_ratio}} --train_data ${{outputs.train_data}} --test_data ${{outputs.test_data}}",
    environment=f"{pipeline_job_env.name}:{pipeline_job_env.version}",
)

data_prep_component = ml_client.create_or_update(data_prep_component.component)
print(f"Component {data_prep_component.name} registered with Version {data_prep_component.version}")
```

---

### **Step 2: Create the Model Training Component**

This component **trains a Gradient Boosting classifier**.

#### **Define the Component**

```python
from azure.ai.ml import load_component

%%writefile {train_src_dir}/train.yml
name: train_credit_defaults_model
display_name: Train Credit Defaults Model
type: command
inputs:
  train_data: type: uri_folder
  test_data: type: uri_folder
  learning_rate: type: number
  registered_model_name: type: string
outputs:
  model: type: uri_folder
code: .
environment: azureml://registries/azureml/environments/sklearn-1.0/labels/latest
command: >-
  python train.py
  --train_data ${{inputs.train_data}}
  --test_data ${{inputs.test_data}}
  --learning_rate ${{inputs.learning_rate}}
  --registered_model_name ${{inputs.registered_model_name}}
  --model ${{outputs.model}}
```

#### **Register the Component**

```python
train_component = load_component(source=os.path.join(train_src_dir, "train.yml"))
train_component = ml_client.create_or_update(train_component)
print(f"Component {train_component.name} registered with Version {train_component.version}")
```

---

## **7. Create the Pipeline**

```python
from azure.ai.ml import dsl

@dsl.pipeline(
    compute="serverless",
    description="Data prep and training pipeline",
)
def credit_defaults_pipeline(data_input, test_train_ratio, learning_rate, model_name):
    data_prep_job = data_prep_component(data=data_input, test_train_ratio=test_train_ratio)
    train_job = train_component(
        train_data=data_prep_job.outputs.train_data,
        test_data=data_prep_job.outputs.test_data,
        learning_rate=learning_rate,
        registered_model_name=model_name,
    )

pipeline = credit_defaults_pipeline(
    data_input=credit_data.path,
    test_train_ratio=0.25,
    learning_rate=0.05,
    model_name="credit_defaults_model",
)
```

---

## **8. Submit and Monitor the Pipeline**

```python
pipeline_job = ml_client.jobs.create_or_update(pipeline, experiment_name="credit_defaults_pipeline")
ml_client.jobs.stream(pipeline_job.name)
```

---

## **Conclusion**

This tutorial **standardizes ML workflows** by:

- **Splitting ML tasks into reusable steps**.
- **Creating and registering ML pipeline components**.
- **Automating data preparation and model training**.

This approach **enhances collaboration, scalability, and efficiency** in ML projects. 🚀
