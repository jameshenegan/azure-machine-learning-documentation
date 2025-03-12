# What is an Azure Machine Learning compute instan

https://learn.microsoft.com/en-us/azure/machine-learning/concept-compute-instance

### **Summary: Azure Machine Learning Compute Instance**

#### **Overview**

An **Azure Machine Learning Compute Instance** is a managed cloud-based workstation tailored for **data scientists** to build, train, and deploy machine learning models. Each compute instance has a **single owner** but allows **file sharing** between instances. It serves as a **fully configured** and **secure development environment** that integrates with Azure Machine Learning (Azure ML).

#### **Key Use Cases**

1. **Development Environment**: Pre-configured workspace with Jupyter, JupyterLab, and VS Code for development.
2. **Training and Inference**: Acts as a compute target for training models and testing inference.
3. **Secure Cloud Workstation**: Managed by enterprise policies, security controls, and networking configurations.

---

## **Benefits of Compute Instance**

### **1. Productivity**

- Integrated with **Azure Machine Learning Studio**.
- Supports **Jupyter, JupyterLab, and VS Code (Preview)**.
- Facilitates **notebook and data sharing** within a workspace.

### **2. Security & Compliance**

- **Enterprise-grade security**, including:
  - **Azure Role-Based Access Control (RBAC)**
  - **Virtual Network Support**
  - **Auto-provisioning** from Azure Resource Manager (ARM) templates.
  - **TLS 1.2 security protocol enabled**.
  - **Option to disable SSH access** for enhanced security.
- No **public IP**, making it **a secure training compute target**.

### **3. Preconfigured for Machine Learning**

- Comes with **pre-installed ML and deep learning packages**.
- Supports **GPU acceleration** and multiple Azure VM types.
- **Customization**: Install additional **libraries, packages, and drivers**.

### **4. Cost Optimization**

- **Auto-start and auto-shutdown** schedules to save costs.
- **Idle shutdown** feature to turn off unused instances.

---

## **Tools & Environments**

Compute Instances come pre-installed with essential tools for **AI/ML development**, including:

### **General Tools**

- **Azure CLI**
- **Docker**
- **Nginx**
- **Blob FUSE** for Azure Storage
- **Intel MPI Library**

### **Python & R Tools**

- **Python Environments**

  - **Anaconda Python**, **Jupyter**, **JupyterLab**, **Azure ML SDK**
  - Deep Learning Packages: **TensorFlow, PyTorch, Keras, Horovod**
  - ONNX Packages: **keras2onnx, skl2onnx, onnxmltools**
  - ML Libraries: **scikit-learn, matplotlib, pandas, tqdm**

- **R Environments**
  - **R Kernel**, support for **RStudio/Posit Workbench**

### **Deep Learning & AI Frameworks**

- **Pre-installed NVIDIA GPU drivers (CUDA, cuDNN)**
- Supports **MLFlow, PyTorch, TensorFlow, ONNX**.

---

## **Accessing & Managing Files**

- **Files & Notebooks** are stored in the **Azure File Share** of the workspace.
- The **mounted storage** allows sharing across multiple compute instances.
- **Best Practices for Storage**:
  - Store notebooks in the file share.
  - **Avoid storing training data on the file share**; use separate storage options.
  - For temporary data, use `/tmp` or `/mnt` (temporary disk is VM-dependent).
  - OS Disk: **120GB** (encrypted with Microsoft-managed keys).

---

## **Creating a Compute Instance**

### **Ways to Create**

- **Azure ML Studio (Notebook Interface)**
- **Azure CLI / Azure ML SDK**
- **Azure Resource Manager Templates**
- **Command-Line Interface (CLI) for Azure ML**

### **Admin Management**

- Admins can create compute instances for **other users**.
- **SSO (Single Sign-On) must be disabled** when creating instances for others.
- **Setup scripts** can be used for automated provisioning.

---

## **Compute Target Capabilities**

A Compute Instance can act as a **training compute target**, similar to **Azure ML Compute Clusters**, but:

- It has **only one node** (single-machine training).
- Supports **multi-GPU distributed training** on a single instance.
- **Job Queue**:
  - Can run **multiple jobs in parallel** (one job per vCPU).
  - Queues additional jobs until resources free up.
- **Inference Testing**: Can be used for local inference before deployment.

---

## **Best Practices & Tips**

- **Storage Management**:
  - If the **OS disk runs out of space**, clear at least **5GB** before rebooting.
  - Temporary disk storage **(/mnt)** is recommended for large temporary data.
- **Stopping & Restarting**:
  - Don’t **manually stop** the compute instance using `sudo shutdown`; use the Azure ML Studio interface instead.
- **Security & Networking**:
  - **Ensure WebSocket connections are enabled** for Jupyter functionalities.

---

## **Conclusion**

Azure Machine Learning Compute Instances provide a **secure, managed, and scalable** development environment for **data scientists and ML engineers**. With **pre-installed tools, enterprise security, and cost-saving features**, they streamline the ML workflow from **development to model deployment**.
