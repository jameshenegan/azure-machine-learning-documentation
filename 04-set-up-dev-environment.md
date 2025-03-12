# Set up a Python development environment for Azure Machine Learning

https://learn.microsoft.com/en-us/azure/machine-learning/how-to-configure-environment

### **Detailed Summary: Setting Up a Python Development Environment for Azure Machine Learning**

This guide explains how to configure a **Python development environment** for Azure Machine Learning (Azure ML). It covers **various development environments**, their **pros and cons**, and step-by-step setup instructions.

---

## **Development Environment Options**

The table below summarizes the three main options:

| **Environment**                         | **Pros**                                                                        | **Cons**                                                            |
| --------------------------------------- | ------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Local Environment**                   | Full control over dependencies, IDEs, and tools.                                | Takes longer to set up. Must install SDK and dependencies manually. |
| **Data Science Virtual Machine (DSVM)** | Pre-installed ML tools (TensorFlow, PyTorch, Scikit-learn, etc.). Easy scaling. | Slower start compared to Azure ML Compute Instance.                 |
| **Azure ML Compute Instance**           | Fastest setup with pre-installed SDK and pre-cloned notebooks.                  | Less control over dependencies. Incurs additional costs.            |

---

## **Prerequisites**

Before setting up your environment, ensure you have:

1. **Azure Machine Learning workspace**
   - If not, create one via:
     - **Azure Portal**
     - **Azure CLI**
     - **Azure Resource Manager templates**
2. **Workspace Configuration File (`config.json`)**

   - Needed for **local environments and DSVM**.
   - This JSON file **connects the SDK to your Azure ML workspace**.
   - Format:
     ```json
     {
       "subscription_id": "<subscription-id>",
       "resource_group": "<resource-group>",
       "workspace_name": "<workspace-name>"
     }
     ```
   - Store it in the **same directory or a subdirectory (`.azureml/`)** as your Python scripts or notebooks.

3. **Python Virtual Environment**
   - Recommended: **Anaconda or Miniconda** for managing dependencies.

---

## **1. Local Computer or Remote VM Setup**

### **Steps:**

1. **Create a virtual environment**
   ```bash
   conda create -n azureml_env python=3.10
   conda activate azureml_env
   ```
2. **Install the Azure ML SDK**
   ```bash
   pip install azure-ai-ml azure-identity
   ```
3. **Connect to Azure ML workspace via Python SDK**

   ```python
   from azure.ai.ml import MLClient
   from azure.identity import DefaultAzureCredential

   subscription_id = '<SUBSCRIPTION_ID>'
   resource_group = '<RESOURCE_GROUP>'
   workspace = '<AZUREML_WORKSPACE_NAME>'

   ml_client = MLClient(DefaultAzureCredential(), subscription_id, resource_group, workspace)
   ```

4. **(Optional) Use Jupyter Notebooks**

   - Install Jupyter Notebook:
     ```bash
     conda install notebook ipykernel
     ```
   - Create a **kernel for your environment**:
     ```bash
     ipython kernel install --user --name azureml_env --display-name "Python (azureml_env)"
     ```
   - Launch Jupyter Notebook:
     ```bash
     jupyter notebook
     ```

5. **Use VS Code for Azure ML**
   - Install **Visual Studio Code**.
   - Install the **Azure Machine Learning extension**.
   - Use VS Code to:
     - **Manage Azure ML resources**
     - **Connect to a remote compute instance**
     - **Debug ML models and deployments**

---

## **2. Azure Machine Learning Compute Instance**

### **Key Features:**

- Secure, **cloud-based Jupyter Notebook server**.
- **Fully managed** ML environment.
- No installation required; just **create a compute instance** inside Azure ML.

### **Steps to Use:**

1. **Create a compute instance** inside your Azure ML workspace.
2. **Start Jupyter Notebook** or use **Azure ML Studio**.
3. **Run ML workloads without additional setup**.

**Tip:** Enable **idle shutdown** to prevent unnecessary charges.

---

## **3. Data Science Virtual Machine (DSVM)**

### **What is DSVM?**

A **pre-configured virtual machine** with:

- ML tools like **TensorFlow, PyTorch, Scikit-learn, XGBoost**.
- Data tools like **Spark Standalone and Drill**.
- **Azure CLI, AzCopy, Storage Explorer**.
- IDEs like **VS Code and PyCharm**.

### **How to Set Up a DSVM**

1. **Create a DSVM using the Azure CLI**
   - **Ubuntu DSVM:**
     ```bash
     az vm create --resource-group YOUR-RESOURCE-GROUP-NAME \
       --name YOUR-VM-NAME \
       --image microsoft-dsvm:linux-data-science-vm-ubuntu:linuxdsvmubuntu:latest \
       --admin-username YOUR-USERNAME \
       --admin-password YOUR-PASSWORD \
       --generate-ssh-keys --authentication-type password
     ```
   - **Windows DSVM:**
     ```bash
     az vm create --resource-group YOUR-RESOURCE-GROUP-NAME \
       --name YOUR-VM-NAME \
       --image microsoft-dsvm:dsvm-windows:server-2016:latest \
       --admin-username YOUR-USERNAME \
       --admin-password YOUR-PASSWORD \
       --authentication-type password
     ```
2. **Set Up a Python Environment for Azure ML**
   - Create a virtual environment:
     ```bash
     conda create -n py310 python=3.10
     conda activate py310
     ```
   - Install the **Azure ML SDK**:
     ```bash
     pip install azure-ai-ml azure-identity
     ```
3. **Connect DSVM to Azure ML Workspace**
   - Use the **workspace configuration file** (`config.json`).
   - Or, initialize directly in Python:
     ```python
     ml_client = MLClient(DefaultAzureCredential(), subscription_id, resource_group, workspace)
     ```
4. **Use DSVM with VS Code**
   - Install the **Azure ML extension** for **VS Code**.
   - Connect DSVM as a **remote compute instance**.

---

## **Comparison of Environment Choices**

| Feature                          | **Local Environment** | **Compute Instance** | **Data Science VM (DSVM)**      |
| -------------------------------- | --------------------- | -------------------- | ------------------------------- |
| **Control over dependencies**    | ✅ Full Control       | ❌ Limited           | ✅ Full Control                 |
| **Pre-installed ML tools**       | ❌ No                 | ✅ Yes               | ✅ Yes                          |
| **Quick Setup**                  | ❌ Longer setup       | ✅ Easiest           | ❌ Slower than Compute Instance |
| **Requires Manual Installation** | ✅ Yes                | ❌ No                | ❌ No                           |
| **Best for**                     | Custom workflows      | Quick experiments    | Scalable data science work      |

---

## **Next Steps**

After setting up your environment, you can:

- **Train and deploy ML models** with the **MNIST dataset**.
- **Explore Azure ML SDK documentation** for more features.

For detailed guidance, refer to:  
➡️ **[Azure Machine Learning SDK for Python reference](https://learn.microsoft.com/en-us/python/api/overview/azure/ml)**.
