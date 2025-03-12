# What are Azure Machine Learning environments?

https://learn.microsoft.com/en-us/azure/machine-learning/concept-environments

### **Detailed Summary of Azure Machine Learning Environments**

Azure Machine Learning **environments** encapsulate the software settings and dependencies required for machine learning training and inference. These environments ensure **reproducibility, portability, and auditability** across different compute targets, allowing users to develop, train, and deploy models in a consistent environment.

---

## **1. Overview of Azure Machine Learning Environments**

An **Azure ML Environment** includes:

- **Python packages**
- **Software dependencies**
- **Base images (Docker/Conda)**
- **Custom configurations**

These environments serve multiple purposes:

- **Develop training scripts** in a defined environment.
- **Reuse environments** for consistent model training.
- **Deploy models** with the same environment used during training.
- **Revisit previous environments** for auditability and reproducibility.

### **Key Benefits**

- **Standardization:** Ensures consistent execution across different compute targets.
- **Versioning:** Keeps track of changes and allows rollback to previous versions.
- **Caching & Reuse:** Speeds up training and deployment by using cached environments.
- **Portability:** Enables moving environments across different workspaces.

---

## **2. Types of Environments**

Azure ML environments are categorized into **Curated, User-Managed, and System-Managed**.

### **1. Curated Environments**

- **Pre-built and managed by Azure ML**.
- **Hosted in the AzureML registry** (maintained by Microsoft).
- Optimized for **fast deployment** and **quick start** with common ML frameworks.
- Cannot be modified **without renaming them**.
- **Examples:** Environments pre-configured with TensorFlow, PyTorch, Scikit-learn.

### **2. User-Managed Environments**

- **Custom environments where users define dependencies manually**.
- Requires setting up **all necessary packages**.
- Can be **shared across workspaces** via a custom **machine learning registry**.
- Supports:
  - **Bring Your Own Container (BYOC)**
  - **Docker Build Context (Azure ML manages image materialization)**.

### **3. System-Managed Environments**

- **Azure ML automatically handles dependency management using Conda**.
- Uses **base Docker images** with a new Conda environment built on top.
- Simplifies environment setup **without requiring manual intervention**.

---

## **3. Creating and Managing Environments**

Users can create and manage environments through:

- **Azure ML Python SDK**
- **Azure ML CLI**
- **Azure ML Studio (GUI)**
- **VS Code Extension**

### **Key Environment Management Capabilities**

| **Feature**                  | **Description**                                                |
| ---------------------------- | -------------------------------------------------------------- |
| **Registering environments** | Save and reuse environment configurations.                     |
| **Fetching environments**    | Retrieve environments for training or deployment.              |
| **Versioning**               | Track changes over time for reproducibility.                   |
| **Editing environments**     | Modify existing environments and create new versions.          |
| **Docker Image Management**  | Automatically build and cache Docker images from environments. |

#### **Anonymous Environments**

- **Automatically created when submitting experiments.**
- **Not listed in workspace but retrievable using their version.**

---

## **4. Environment Building, Caching, and Reuse**

Azure ML builds **environment definitions into Docker images**, caching them for reuse in subsequent jobs.

### **Workflow for Using Environments**

1. **Submit a job** with an environment → Azure ML **builds a Docker image**.
2. **The image is cached** in the workspace’s **container registry**.
3. **For subsequent jobs, Azure ML reuses the cached image** (if unchanged).
4. **If the environment definition changes, a new image is built**.

### **Building Environments as Docker Images**

| **Step**                | **System-Managed**       | **User-Managed**                              |
| ----------------------- | ------------------------ | --------------------------------------------- |
| **Base Image**          | Downloaded automatically | User-defined                                  |
| **Custom Docker Steps** | Automated by Azure ML    | User-defined                                  |
| **Python Packages**     | Installed via Conda      | Pre-installed in base image or added manually |

### **Image Caching & Reuse**

- **Azure ML uses a hash function** to check whether an environment already exists.
- **Hash includes:**
  - Base image
  - Docker steps
  - Python packages
- **Same hash → Cached image is reused**.
- **New hash → Triggers a new image build**.

#### **Example: Environment Hashing**

| **Environment** | **Base Image** | **Python Packages** | **Hash Value** | **Cached Image Used?** |
| --------------- | -------------- | ------------------- | -------------- | ---------------------- |
| Env 1           | Ubuntu 18.04   | numpy, pandas       | `123abc`       | ✅ Yes                 |
| Env 2           | Ubuntu 18.04   | numpy, pandas       | `123abc`       | ✅ Yes                 |
| Env 3           | Ubuntu 18.04   | numpy, scipy        | `456def`       | ❌ No (New Image)      |

**Key Considerations:**

- **Renaming an environment doesn't change its hash.**
- **Changing package versions, dependencies, or base images changes the hash.**

---

## **5. Environment Best Practices**

### **Pinned vs. Unpinned Dependencies**

- **Unpinned dependencies (e.g., `numpy`)** → Uses the latest version at creation time.
- **Pinned dependencies (`numpy==1.18.1`)** → Ensures version consistency.
- **Best Practice:** Pin dependencies to avoid unexpected changes.

### **Handling Base Image Updates**

- Using an **unpinned base image (e.g., `mcr.microsoft.com/azureml/openmpi3.1.2-ubuntu18.04`)** triggers a rebuild when updated.
- Ensures security patches and updates.
- **Best Practice:** Use a specific version tag if stability is required.

---

## **6. Image Patching & Security**

### **Microsoft's Patch Commitments**

- Patches for **known security vulnerabilities** are released **every two weeks**.
- **No unpatched vulnerabilities older than 30 days** in the latest images.
- New patched images receive a **new immutable tag**, updating the `:latest` tag.

### **Updating Environments with New Patches**

- **Managed Online Endpoints:** Need redeployment to apply patched images.
- **Custom Images:** Users are responsible for **applying security updates manually**.

---

## **7. Key Takeaways**

1. **Azure ML Environments** define **dependencies and configurations** for training and inference.
2. **Three types of environments:**
   - **Curated (Pre-built, managed by Microsoft)**
   - **User-Managed (Fully customized, BYOC/Docker)**
   - **System-Managed (Conda-based, automated by Azure ML)**
3. **Environments are managed via Azure ML SDK, CLI, and Studio** for easy creation, versioning, and tracking.
4. **Azure ML builds Docker images from environments**, caching them to speed up subsequent jobs.
5. **Environment hashing ensures reuse**—if dependencies remain unchanged, cached images are used.
6. **Unpinned dependencies and base images can trigger unexpected updates**, while pinned versions provide stability.
7. **Microsoft patches base images every two weeks** for security, but custom images require **manual updates**.
8. **Best practices include:**
   - Pin dependencies for consistency.
   - Use environment versioning.
   - Regularly update images to apply security patches.

By leveraging Azure Machine Learning environments efficiently, users can **ensure reproducibility, improve deployment speed, and maintain secure machine learning workflows**.
