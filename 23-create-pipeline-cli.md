# Create and run machine learning pipelines using components with the Azure Machine Learning CLI

https://learn.microsoft.com/en-us/azure/machine-learning/how-to-create-component-pipelines-cli

### **Detailed Summary: Create and Run Machine Learning Pipelines using Components with Azure Machine Learning CLI**

## **Overview**

This article provides a step-by-step guide on how to **build and run Azure Machine Learning (ML) pipelines** using the **Azure CLI (Command Line Interface)** and YAML-based components. The Azure ML CLI allows you to define, run, and manage machine learning pipelines without needing to write extensive Python code.

🔹 **Why use components?**

- **Modularity** – Each pipeline step is reusable.
- **Scalability** – Run pipelines across different compute clusters.
- **Automation** – Easily automate ML workflows.

🔹 **Pipeline Creation Methods in Azure ML**

- **CLI (Command Line Interface) – Focus of this article** 🏆
- **Python SDK**
- **Azure ML Studio (Drag-and-Drop UI)**

---

## **1. Prerequisites**

Before creating a pipeline, ensure you have:

- **Azure subscription** (free or paid).
- **Azure Machine Learning workspace**.
- **Azure CLI installed with the Machine Learning extension**.
- **Cloned Azure ML examples repository:**
  ```bash
  git clone https://github.com/Azure/azureml-examples --depth 1
  cd azureml-examples/cli/jobs/pipelines-with-components/basics
  ```

---

## **2. Create Your First Pipeline with Components**

A pipeline consists of multiple **components**, where each component performs a specific ML task (data processing, training, evaluation, etc.).

🔹 **Pipeline Structure (Example)**

1. **Data Processing** – Prepare raw data.
2. **Model Training** – Train the ML model.
3. **Model Evaluation** – Test and validate the model.

🔹 **Pipeline Directory Structure**

```
3b_pipeline_with_data/
│── pipeline.yml            # Defines the pipeline workflow
│── componentA.yml          # Defines Component A
│── componentB.yml          # Defines Component B
│── componentC.yml          # Defines Component C
└── component_src/          # Source code for the components
```

---

### **2.1 Set Up Compute Cluster**

First, check available compute resources:

```bash
az ml compute list
```

If no compute is available, create a cluster:

```bash
az ml compute create -n cpu-cluster --type amlcompute --min-instances 0 --max-instances 10
```

⚡ _Note:_ Skip this step if using **serverless compute**.

---

### **2.2 Run a Pipeline Job**

Submit the pipeline job using:

```bash
az ml job create --file pipeline.yml
```

After submission, Azure ML returns a JSON response with:

- **Job Name** (GUID-based identifier)
- **Experiment Name**
- **Status** (e.g., `Preparing`)
- **Monitoring URL** _(view pipeline graph in Azure ML Studio)_

To **visualize the pipeline**, open the `services.Studio.endpoint` URL.

---

## **3. Understand Pipeline Definition (pipeline.yml)**

### **3.1 Pipeline YAML Breakdown**

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/pipelineJob.schema.json
type: pipeline

display_name: 3b_pipeline_with_data
description: Pipeline with 3 component jobs with data dependencies

settings:
  default_compute: azureml:cpu-cluster # Set default compute cluster

outputs:
  final_pipeline_output:
    mode: rw_mount # Mount for read/write

jobs:
  component_a:
    type: command
    component: ./componentA.yml
    inputs:
      component_a_input:
        type: uri_folder
        path: ./data
    outputs:
      component_a_output:
        mode: rw_mount # Output mounted for further steps

  component_b:
    type: command
    component: ./componentB.yml
    inputs:
      component_b_input: ${{parent.jobs.component_a.outputs.component_a_output}}
    outputs:
      component_b_output:
        mode: rw_mount

  component_c:
    type: command
    component: ./componentC.yml
    inputs:
      component_c_input: ${{parent.jobs.component_b.outputs.component_b_output}}
    outputs:
      component_c_output: ${{parent.outputs.final_pipeline_output}}
```

🔹 **Key Highlights:**

- **Pipeline Type:** Defined as `type: pipeline`.
- **Compute Resource:** Uses `default_compute: azureml:cpu-cluster`.
- **Pipeline Steps (`jobs`)**
  - **component_a:** Reads input data.
  - **component_b:** Takes `component_a`’s output.
  - **component_c:** Takes `component_b`’s output.
- **Data Dependencies**: Output from one step is passed as an input to the next using `${{parent.jobs.<component>.outputs.<output_name>}}`.

⚡ _Note:_ If using **serverless compute**, replace:

```yaml
default_compute: azureml:cpu-cluster
```

with:

```yaml
default_compute: azureml:serverless
```

---

## **4. Understand Component Definition (componentA.yml)**

Each **component** is a reusable ML step with defined **inputs, outputs, execution command, and environment**.

### **4.1 Example: Component YAML (componentA.yml)**

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/commandComponent.schema.json
type: command

name: component_a
display_name: Component A
version: 1

inputs:
  component_a_input:
    type: uri_folder

outputs:
  component_a_output:
    type: uri_folder

code: ./componentA_src

environment:
  image: python # Uses a predefined Python environment

command: >-
  python hello.py --componentA_input ${{inputs.component_a_input}} --componentA_output ${{outputs.component_a_output}}
```

🔹 **Key Highlights:**

- **`inputs`**: Defines input data type (`uri_folder` for directories).
- **`outputs`**: Defines output type.
- **`command`**: The shell command to execute.
- **`environment`**: Defines execution environment (Docker image, Conda, or ML environment).

💡 _Inside `hello.py`, command-line arguments are parsed to process the input data and generate output._

---

## **5. Register Components for Reuse**

Once components are created, they can be **registered** in the Azure ML workspace for reuse.

### **5.1 Register a Component**

```bash
az ml component create --file train.yml
az ml component create --file score.yml
az ml component create --file eval.yml
```

💡 \*After registration, components appear in **Azure ML Studio > Assets > Components\***.

---

## **6. Use Registered Components in a Pipeline**

Instead of defining components in a local YAML file, **use registered components**.

### **6.1 Example: Pipeline Using Registered Components**

```yaml
type: command
component: azureml:my_train@latest # Uses latest registered version

inputs:
  training_data:
    type: uri_folder
    path: ./data
  max_epochs: ${{parent.inputs.pipeline_max_epochs}}
  learning_rate: ${{parent.inputs.pipeline_learning_rate}}
outputs:
  model_output: ${{parent.outputs.trained_model}}
```

🔹 **Benefits of Registered Components:**

- **Reusability**: Use the same component across multiple pipelines.
- **Version Control**: Older versions remain available.

---

## **7. Managing Components via CLI**

Use **Azure ML CLI** to **list, update, or delete components**.

### **7.1 List Registered Components**

```bash
az ml component list
```

### **7.2 Show Component Details**

```bash
az ml component show --name component_a --version 1
```

### **7.3 Archive (Delete) a Component**

```bash
az ml component archive --name component_a --version 1
```

---

## **8. Next Steps**

- **[Try CLI v2 Component Example](https://github.com/Azure/azureml-examples)**
- **[Azure ML CLI Reference](https://learn.microsoft.com/azure/machine-learning/how-to-cli-v2)**
- **[Deploy ML Pipelines](https://learn.microsoft.com/azure/machine-learning/how-to-deploy-pipeline)**

---

## **🚀 Summary**

✅ **Azure ML CLI** allows you to create **scalable and reusable pipelines** using **YAML components**.  
✅ **Components** define modular steps, making ML workflows **flexible and efficient**.  
✅ **Registered components** enable **versioning** and **reuse** across projects.  
✅ **Pipelines** can be run using **serverless compute** or dedicated **CPU/GPU clusters**.

By leveraging **Azure ML CLI and YAML**, you can build **end-to-end ML workflows** with minimal code and maximum **automation**! 🚀
