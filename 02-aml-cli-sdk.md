# What is Azure Machine Learning CLI and Python SDK v2?

https://learn.microsoft.com/en-us/azure/machine-learning/concept-v2

### **Detailed Summary: Azure Machine Learning CLI and Python SDK v2**

Azure Machine Learning CLI v2 and Python SDK v2 provide a **consistent interface and terminology** across Azure ML tools. They introduce **significant syntax changes** compared to v1, enhancing usability, automation, and compatibility. While the CLI is better suited for **CI/CD and automation**, the SDK is ideal for **development and scripting**.

---

## **Azure Machine Learning CLI v2**

### **Overview**

Azure Machine Learning CLI v2 is the **latest extension for Azure CLI** that enables users to create and manage ML assets and workflows via command-line commands. Workflows and assets are defined using **YAML configuration files**, specifying what the asset is and where it runs.

### **Key Features**

- Uses the command structure:
  ```shell
  az ml <noun> <verb> <options>
  ```
- Assets are configured using **YAML files**.
- Commands allow interaction with various ML components:
  ```shell
  az ml job create --file my_job_definition.yaml
  az ml environment update --name my-env --file my_updated_env_definition.yaml
  az ml model list
  az ml compute show --name my_compute
  ```

### **Use Cases**

1. **Easy Onboarding**

   - Users can work with ML models without needing a specific programming language.
   - Supports **Python, R, Java, Julia, and C#** for custom logic.
   - YAML files define ML workflows, separating them from script files.

2. **Simplified Deployment & Automation**

   - CLI v2 enables seamless execution from any platform.
   - Useful for **CI/CD pipelines** and **MLOps workflows**.

3. **Managed Inferencing**

   - Supports **real-time and batch inference deployments** through endpoints.

4. **Reusable Pipeline Components**
   - Allows users to **manage and reuse** common logic across ML pipelines.

---

## **Azure Machine Learning Python SDK v2**

### **Overview**

Azure Machine Learning Python SDK v2 is an **updated Python SDK** that provides Python-based interaction with Azure ML for training, managing data/models, and deploying inferencing solutions.

### **Key Features**

- Supports training job submission, data and model management, and inferencing.
- Enables **machine learning pipeline creation**.
- Commands maintain **parity with CLI v2**, ensuring consistency.
- Example of listing assets:
  ```python
  from azure.ai.ml import MLClient
  ml_client.models.list()
  ```

### **Use Cases**

1. **Python-Based ML Development**

   - Users can construct **single commands** or **complex workflows** using Python functions.
   - Example:
     ```python
     job = command(name="train_model", inputs=training_data)
     result = job.run()
     ```

2. **Incremental Workflow Complexity**

   - Start with a simple command.
   - Add **hyperparameter tuning**.
   - Expand into a **multi-step pipeline**.

3. **Reusable Pipeline Components**

   - Facilitates **modular ML development**.
   - Promotes reuse of **predefined components** across multiple ML workflows.

4. **Managed Inferencing**
   - Supports **real-time and batch inference** for streamlined deployment.

---

## **Choosing Between v1 and v2**

### **CLI v2**

- **CLI v1 will be deprecated on September 30, 2025**.
- Users must **migrate to CLI v2** before that date.

### **SDK v2**

- SDK v1 **does not have a deprecation date** yet.
- Consider migrating to **SDK v2** if:
  - You need **new features** like **reusable components** and **managed inferencing**.
  - You're starting a **new ML workflow or pipeline**.
  - You want a **more intuitive Python API** for job and pipeline management.

---

## **Conclusion**

Both Azure Machine Learning CLI v2 and Python SDK v2 provide **feature parity** and introduce **enhanced usability**. CLI v2 is **ideal for automation, CI/CD, and MLOps**, while SDK v2 is **better suited for ML development using Python**. Users should **migrate from v1 to v2** to benefit from **new capabilities and long-term support**.
