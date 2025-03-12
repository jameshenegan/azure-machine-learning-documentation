# Expressions in Azure Machine Learning SDK and CLI v2

https://learn.microsoft.com/en-us/azure/machine-learning/concept-expressions

### **Detailed Summary: Expressions in Azure Machine Learning SDK and CLI v2**

Azure Machine Learning SDK and CLI v2 provide **expressions** to reference values dynamically in **jobs** or **components** when their values may not be known at the time of authoring. These expressions are resolved either:

- **On the client-side** (where the job is submitted, e.g., local machine or compute instance).
- **On the server-side** (when the job runs in Azure ML).

### **Expression Syntax**

Expressions use the following format:

```plaintext
${{ <expression> }}
```

This syntax allows placeholders to be substituted at runtime with actual values.

---

## **1. Client-Side Expressions**

These expressions are evaluated **before** the job runs, at the time of job submission. They resolve on the **local client machine** or **compute instance**.

| **Expression**                                       | **Description**                                                                                                                | **Scope**               |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ----------------------- |
| `${{inputs.<input_name>}}`                           | References an **input** data asset or model.                                                                                   | Works for all jobs.     |
| `${{outputs.<output_name>}}`                         | References an **output** data asset or model.                                                                                  | Works for all jobs.     |
| `${{search_space.<hyperparameter>}}`                 | References a **hyperparameter** in a sweep job. The hyperparameter value for each trial is selected based on the search space. | **Sweep jobs only**.    |
| `${{parent.inputs.<input_name>}}`                    | Binds the **inputs** of a child job (pipeline step) to the **inputs** of the parent pipeline job.                              | **Pipeline jobs only**. |
| `${{parent.outputs.<output_name>}}`                  | Binds the **outputs** of a child job (pipeline step) to the **outputs** of the parent pipeline job.                            | **Pipeline jobs only**. |
| `${{parent.jobs.<step-name>.inputs.<input-name>}}`   | Binds to the **inputs** of another step in the pipeline.                                                                       | **Pipeline jobs only**. |
| `${{parent.jobs.<step-name>.outputs.<output-name>}}` | Binds to the **outputs** of another step in the pipeline.                                                                      | **Pipeline jobs only**. |

### **Example: Using Client Expressions in a Training Job**

```yaml
inputs:
  training_data: ${{inputs.training_data}}
  learning_rate: ${{search_space.learning_rate}}
outputs:
  model_output: ${{outputs.model_output}}
```

This example:

- Uses `${{inputs.training_data}}` to reference input data.
- Uses `${{search_space.learning_rate}}` to get hyperparameter values for tuning.
- Uses `${{outputs.model_output}}` to define the output location.

---

## **2. Server-Side Expressions**

These expressions are evaluated **when the job runs on Azure ML** (server-side). This means:

- They use the **current state of the workspace** at job execution time.
- If a **scheduled job** is created and the workspace state changes (e.g., default datastore changes), the expression resolves to the **new workspace state** at execution time.

| **Expression**           | **Description**                                                                 | **Scope**           |
| ------------------------ | ------------------------------------------------------------------------------- | ------------------- |
| `${{default_datastore}}` | Resolves to the **default datastore** for the workspace or pipeline.            | Works for all jobs. |
| `${{name}}`              | The **job name** (for pipelines, the step job name, not the pipeline job name). | Works for all jobs. |
| `${{output_name}}`       | The **output name** for the job.                                                | Works for all jobs. |

### **Example: Using Server Expressions in Output Paths**

```yaml
outputs:
  model_output:
    path: azureml://datastores/${{default_datastore}}/paths/${{name}}/${{output_name}}
```

At runtime, this resolves to:

```plaintext
azureml://datastores/workspaceblobstore/paths/<job-name>/model_path
```

If the workspace's **default datastore** changes, the expression automatically resolves to the **updated** datastore.

---

## **3. Key Differences Between Client and Server Expressions**

| **Aspect**                      | **Client Expressions**                              | **Server Expressions**                            |
| ------------------------------- | --------------------------------------------------- | ------------------------------------------------- |
| **When Evaluated**              | Before job submission (on the local machine)        | When the job runs on Azure ML                     |
| **Example**                     | `${{inputs.training_data}}`                         | `${{default_datastore}}`                          |
| **Effect of Workspace Changes** | Uses the state at job submission                    | Uses the latest workspace state when the job runs |
| **Common Use Cases**            | Input/output references, pipeline step dependencies | Dynamic paths, default datastore references       |

---

## **4. Use Cases and Best Practices**

- **Use Client Expressions for Input/Output References**
  - Example: `${{inputs.dataset_path}}`
  - Best for **defining datasets**, **hyperparameters**, and **component dependencies**.
- **Use Server Expressions for Dynamic Resources**

  - Example: `${{default_datastore}}`
  - Useful when **workspaces change over time**, such as changing **default datastores**.

- **Use Parent Expressions for Pipeline Jobs**
  - Example: `${{parent.inputs.input_data}}`
  - Ensures that **child jobs inherit the correct data**.

---

## **5. Next Steps**

To learn more about expressions in Azure ML SDK and CLI v2, refer to:

- **[CLI v2 core YAML syntax](https://learn.microsoft.com/azure/machine-learning/how-to-cli-v2)**
- **[Hyperparameter tuning a model](https://learn.microsoft.com/azure/machine-learning/how-to-tune-hyperparameters)**
- **[Tutorial: ML pipelines with Python SDK v2](https://learn.microsoft.com/azure/machine-learning/how-to-pipeline-sdk-v2)**
- **[Example: Iris batch prediction notebook](https://github.com/Azure/azureml-examples/blob/main/sdk/python/jobs/pipelines/iris_batch_prediction.ipynb)**
- **[Example: Pipeline YAML file](https://learn.microsoft.com/azure/machine-learning/how-to-pipeline-cli-v2)**

---

## **Conclusion**

✅ **Expressions** in Azure ML allow dynamic referencing of values.  
✅ **Client expressions** are evaluated **before submission** (local).  
✅ **Server expressions** are resolved **at runtime** (Azure ML).  
✅ **Use case-based expressions** help in designing **efficient ML workflows**.

By using **expressions strategically**, you can **automate workflows, optimize pipelines, and ensure dynamic job execution** in **Azure Machine Learning**. 🚀
