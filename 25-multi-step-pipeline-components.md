# Use multistep pipeline components in pipeline jobs

https://learn.microsoft.com/en-us/azure/machine-learning/how-to-use-pipeline-component

### **Detailed Summary: Using Multistep Pipeline Components in Azure Machine Learning**

## **Overview**

Azure Machine Learning (ML) pipelines often involve complex workflows that require multiple steps, such as **data preprocessing, model training, and evaluation**. To streamline development, you can group multiple steps into **a single pipeline component** and reuse it across different ML workflows.

This guide explains how to:

- **Nest multiple steps** inside a pipeline component.
- **Define, manage, and use multistep pipeline components**.
- **Leverage pipeline components to enhance reusability and collaboration**.

---

## **1. What Are Multistep Pipeline Components?**

Multistep **pipeline components** allow you to **group multiple related steps** in a machine learning pipeline into a single reusable component.

✅ **Benefits**:

- **Reusability**: Define once, reuse across different pipelines.
- **Modularity**: Develop and test subtasks independently.
- **Scalability**: Assign different compute resources to different steps.
- **Abstraction**: Users don’t need to understand the internal details of each step.

🔹 **Pipeline Components vs. Pipeline Jobs**:
| Feature | **Pipeline Component** | **Pipeline Job** |
|----------|-----------------|----------------|
| **Definition** | Contains a group of steps | Contains components and runtime settings |
| **Inputs & Outputs** | Defined but not assigned values | Assigned values at runtime |
| **Compute & Data Settings** | Not hardcoded, assigned at runtime | Can define default compute/data settings |

---

## **2. Prerequisites**

Before building multistep pipeline components, ensure you have:

- **An Azure ML workspace**: Follow the [Create resources](https://learn.microsoft.com/en-us/azure/machine-learning/concept-workspace) guide.
- **Knowledge of Azure ML pipelines & components**.
- **Azure CLI and ML extension installed**.
- **Familiarity with CLI v2 for ML pipeline execution**.

---

## **3. Building a Multistep Pipeline Component**

Multistep pipeline components are defined using YAML. The following example demonstrates:

- A **pipeline component** that contains multiple steps.
- A **pipeline job** that references the component.

### **3.1 Define a Multistep Pipeline Component**

This **pipeline component** includes:

- **Training (`train_job`)**
- **Scoring (`score_job`)**
- **Evaluation (`evaluate_job`)**

#### **Pipeline Component YAML (`train_pipeline_component.yml`)**

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/pipelineComponent.schema.json
type: pipeline

name: train_pipeline_component
display_name: train_pipeline_component
description: Train-score-eval pipeline component

inputs:
  training_data:
    type: uri_folder
  test_data:
    type: uri_folder
  training_max_epochs:
    type: integer
  training_learning_rate:
    type: number
  learning_rate_schedule:
    type: string
    default: "time-based"
  train_node_compute:
    type: string # Promotes compute selection to pipeline level

outputs:
  trained_model:
    type: uri_folder
  evaluation_report:
    type: uri_folder

jobs:
  train_job:
    type: command
    component: ./train/train.yml
    inputs:
      training_data: ${{parent.inputs.training_data}}
      max_epochs: ${{parent.inputs.training_max_epochs}}
      learning_rate: ${{parent.inputs.training_learning_rate}}
      learning_rate_schedule: ${{parent.inputs.learning_rate_schedule}}
    outputs:
      model_output: ${{parent.outputs.trained_model}}
    compute: ${{parent.inputs.train_node_compute}}

  score_job:
    type: command
    component: ./score/score.yml
    inputs:
      model_input: ${{parent.jobs.train_job.outputs.model_output}}
      test_data: ${{parent.inputs.test_data}}
    outputs:
      score_output:
        mode: upload

  evaluate_job:
    type: command
    component: ./eval/eval.yml
    inputs:
      scoring_result: ${{parent.jobs.score_job.outputs.score_output}}
    outputs:
      eval_output: ${{parent.outputs.evaluation_report}}
```

**Key Highlights:**

- The pipeline **receives inputs** (data, hyperparameters, compute).
- Steps **pass outputs to downstream steps**.
- **Compute selection (`train_node_compute`)** is promoted as a pipeline-level input.

---

## **4. Using Multistep Components in Pipeline Jobs**

A **pipeline job** references the **multistep pipeline component** as a single reusable unit.

#### **Pipeline Job YAML (`pipeline.yml`)**

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/pipelineJob.schema.json

display_name: pipeline_with_pipeline_component
experiment_name: pipeline_with_pipeline_component
description: Train and compare models with different learning rates
type: pipeline

inputs:
  pipeline_job_training_data:
    type: uri_folder
    path: ./data
  pipeline_job_test_data:
    type: uri_folder
    path: ./data
  pipeline_job_training_learning_rate1: 0.1
  pipeline_job_training_learning_rate2: 0.01
  compute_train_node: cpu-cluster
  compute_compare_node: cpu-cluster

outputs:
  pipeline_job_best_model:
    mode: upload
  pipeline_job_best_result:
    mode: upload

settings:
  default_datastore: azureml:workspaceblobstore
  default_compute: azureml:cpu-cluster
  continue_on_step_failure: false

jobs:
  train_and_evaluate_model1:
    type: pipeline
    component: ./components/train_pipeline_component.yml
    inputs:
      training_data: ${{parent.inputs.pipeline_job_training_data}}
      test_data: ${{parent.inputs.pipeline_job_test_data}}
      training_max_epochs: 20
      training_learning_rate: ${{parent.inputs.pipeline_job_training_learning_rate1}}
      train_node_compute: ${{parent.inputs.compute_train_node}}

  train_and_evaluate_model2:
    type: pipeline
    component: ./components/train_pipeline_component.yml
    inputs:
      training_data: ${{parent.inputs.pipeline_job_training_data}}
      test_data: ${{parent.inputs.pipeline_job_test_data}}
      training_max_epochs: 20
      training_learning_rate: ${{parent.inputs.pipeline_job_training_learning_rate2}}
      train_node_compute: ${{parent.inputs.compute_train_node}}

  compare:
    type: command
    component: ./components/compare2/compare2.yml
    compute: ${{parent.inputs.compute_compare_node}}
    inputs:
      model1: ${{parent.jobs.train_and_evaluate_model1.outputs.trained_model}}
      eval_result1: ${{parent.jobs.train_and_evaluate_model1.outputs.evaluation_report}}
      model2: ${{parent.jobs.train_and_evaluate_model2.outputs.trained_model}}
      eval_result2: ${{parent.jobs.train_and_evaluate_model2.outputs.evaluation_report}}
    outputs:
      best_model: ${{parent.outputs.pipeline_job_best_model}}
      best_result: ${{parent.outputs.pipeline_job_best_result}}
```

### **Key Highlights**

- **Two model training jobs** run with different learning rates.
- **A comparison step** evaluates which trained model is better.
- **Pipeline components act as reusable sub-pipelines**.

---

## **5. Registering & Reusing Pipeline Components**

To share and reuse pipeline components, you must register them.

#### **Register a Component via Azure CLI**

```bash
az ml component create --file train_pipeline_component.yml
```

#### **List Registered Components**

```bash
az ml component list
```

#### **Use Registered Components in Pipelines**

Modify `pipeline.yml` to reference registered components:

```yaml
component: azureml:train_pipeline_component@latest
```

---

## **6. Summary & Benefits**

✅ **Pipeline components simplify complex ML workflows**.  
✅ **Modular approach improves collaboration & reusability**.  
✅ **Promoting compute as a pipeline input enables flexible execution**.  
✅ **Pipeline jobs reference components just like other steps**.  
✅ **Components can be registered & reused across different projects**.

By leveraging **multistep pipeline components**, teams can efficiently develop, share, and scale ML workflows within **Azure Machine Learning**. 🚀
