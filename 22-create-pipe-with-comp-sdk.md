# Create and run machine learning pipelines using components with the Azure Machine Learning SDK v2

https://learn.microsoft.com/en-us/azure/machine-learning/how-to-create-component-pipeline-python

### **Detailed Summary: Create and Run Machine Learning Pipelines using Components with the Azure Machine Learning SDK v2**

## **Overview**

This article explains how to **build, execute, and manage machine learning pipelines** using the **Azure Machine Learning SDK v2**. The example demonstrates an **image classification task** using the **Fashion MNIST dataset**, where the pipeline consists of **three steps**:

1. **Prepare data** – Convert dataset into CSV format.
2. **Train the model** – Train a **Keras convolutional neural network (CNN)**.
3. **Score the model** – Evaluate the trained model on test data.

By using **Azure Machine Learning pipelines**, you can:
✅ **Automate workflows**  
✅ **Enhance reusability**  
✅ **Optimize resource allocation**  
✅ **Simplify collaboration**

---

## **1. Prerequisites**

Before starting, ensure that you have:

- **Azure Machine Learning workspace** (if not, create one).
- **Python environment** with **Azure ML SDK v2 installed**.
- **Cloned Azure ML examples repository**:
  ```bash
  git clone --depth 1 https://github.com/Azure/azureml-examples
  cd azureml-examples/sdk
  ```

---

## **2. Define the Machine Learning Pipeline**

A machine learning pipeline is built using **components**, each representing a self-contained **step**. The **Fashion MNIST dataset** contains **28x28 grayscale images of clothing** categorized into **10 classes**.

**Pipeline structure:**

1. **Prepare data** (convert images to CSV format).
2. **Train model** (train a CNN for classification).
3. **Score model** (evaluate model on test data).

---

## **3. Create Components**

Each step in the pipeline is defined as a **component**, which includes:

- **Python script** (execution logic).
- **Interface definition** (inputs, outputs).
- **Metadata** (name, version, description).
- **Execution environment** (Docker image, Conda dependencies).

---

### **3.1 Prepare Data Component**

The first step is to prepare the data by converting **Fashion MNIST** files into CSV format.

#### **Component Definition (Python function)**

```python
import os
from pathlib import Path
from mldesigner import command_component, Input, Output

@command_component(
    name="prep_data",
    version="1",
    display_name="Prep Data",
    description="Convert data to CSV format for training and testing",
    environment=dict(
        conda_file=Path(__file__).parent / "conda.yaml",
        image="mcr.microsoft.com/azureml/openmpi4.1.0-ubuntu20.04",
    ),
)
def prepare_data_component(
    input_data: Input(type="uri_folder"),
    training_data: Output(type="uri_folder"),
    test_data: Output(type="uri_folder"),
):
    # Function to convert images to CSV format
    convert(input_data, training_data, test_data)
```

#### **Conda Environment (conda.yaml)**

```yaml
name: imagekeras_prep_conda_env
channels:
  - defaults
dependencies:
  - python=3.7.11
  - pip=20.0
  - pip:
      - mldesigner==0.1.0b4
```

---

### **3.2 Train Model Component**

The second step trains a **CNN model** on the preprocessed dataset.

#### **Component Definition (Python function)**

```python
from mldesigner import command_component, Input, Output

@command_component(
    name="train_image_classification_keras",
    version="1",
    display_name="Train Image Classification Model",
    description="Train a Keras CNN model for image classification",
    environment=dict(
        conda_file=Path(__file__).parent / "conda.yaml",
        image="mcr.microsoft.com/azureml/openmpi4.1.0-ubuntu20.04",
    ),
)
def keras_train_component(
    input_data: Input(type="uri_folder"),
    output_model: Output(type="uri_folder"),
    epochs=10,
):
    from train import train  # Import training logic
    train(input_data, output_model, epochs)
```

#### **Conda Environment (conda.yaml)**

```yaml
name: imagekeras_train_conda_env
channels:
  - defaults
dependencies:
  - python=3.8
  - pip=20.2
  - pip:
      - tensorflow==2.7.0
      - numpy==1.21.4
      - scikit-learn==1.0.1
      - pandas==1.3.4
      - matplotlib==3.2.2
```

---

### **3.3 Score Model Component**

The third step evaluates the trained model.

#### **Component Definition (YAML)**

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/commandComponent.schema.json
type: command

name: score_image_classification_keras
display_name: Score Image Classification Model
inputs:
  input_data:
    type: uri_folder
  input_model:
    type: uri_folder
outputs:
  output_result:
    type: uri_folder
code: ./
command: python score.py --input_data ${{inputs.input_data}} --input_model ${{inputs.input_model}} --output_result ${{outputs.output_result}}
environment:
  conda_file: ./conda.yaml
  image: mcr.microsoft.com/azureml/openmpi4.1.0-ubuntu20.04
```

---

## **4. Load Components & Build the Pipeline**

### **4.1 Load Components**

```python
from prep.prep_component import prepare_data_component
from train.train_component import keras_train_component
from azure.ai.ml import load_component

keras_score_component = load_component(source="./score/score.yaml")
```

### **4.2 Define the Pipeline**

```python
from azure.ai.ml.dsl import pipeline

@pipeline(default_compute="cpu-cluster")
def image_classification_pipeline(pipeline_input_data):
    """Pipeline for image classification using Keras CNN."""
    prepare_data_node = prepare_data_component(input_data=pipeline_input_data)

    train_node = keras_train_component(
        input_data=prepare_data_node.outputs.training_data
    )
    train_node.compute = "gpu-cluster"

    score_node = keras_score_component(
        input_data=prepare_data_node.outputs.test_data,
        input_model=train_node.outputs.output_model,
    )

# Create pipeline instance
pipeline_job = image_classification_pipeline(pipeline_input_data="fashion_ds")
```

---

## **5. Submit & Monitor the Pipeline**

### **5.1 Connect to Azure ML Workspace**

```python
from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential

ml_client = MLClient.from_config(credential=DefaultAzureCredential())
```

### **5.2 Submit Pipeline Job**

```python
pipeline_job = ml_client.jobs.create_or_update(
    pipeline_job, experiment_name="pipeline_samples"
)
```

### **5.3 Monitor Pipeline Progress**

```python
ml_client.jobs.stream(pipeline_job.name)
```

- First-time runs may take **15+ minutes** (due to dependency setup).
- Subsequent runs are **faster** due to caching.

---

## **6. Review & Debug Pipeline in Azure ML Studio**

- Open **Azure Machine Learning Studio**.
- Navigate to **Experiments > pipeline_samples**.
- Check **logs, outputs, and debugging information**.

---

## **7. Register Components for Reuse**

You can **register components** to make them reusable across different pipelines.

```python
try:
    prep = ml_client.components.get(name="prep_data", version="1")
except:
    prep = ml_client.components.create_or_update(prepare_data_component)

# List registered components
for c in ml_client.components.list():
    print(c)
```

---

## **8. Next Steps**

- **[Azure ML SDK Examples](https://github.com/Azure/azureml-examples)** – More real-world ML pipelines.
- **[Azure ML CLI Guide](https://learn.microsoft.com/azure/machine-learning/how-to-cli-v2)** – Create pipelines using CLI.
- **[Deploy Pipelines as Batch Endpoints](https://learn.microsoft.com/azure/machine-learning/how-to-deploy-pipeline)**.

---

### **🚀 Summary**

✅ **Azure ML Pipelines** automate ML workflows for **scalability & efficiency**.  
✅ Components are **modular, reusable, and version-controlled**.  
✅ Use **Python SDK v2** to **define, submit, and monitor** ML pipelines.  
✅ The pipeline can be **executed on CPU or GPU clusters** for optimization.  
✅ **Register components** for sharing and reuse across projects.

By implementing **Azure ML pipelines**, ML workflows become **faster, automated, and production-ready**! 🚀
