# Git integration for Azure Machine Learning

https://learn.microsoft.com/en-us/azure/machine-learning/concept-train-model-git-integration?view=azureml-api-2

### **Detailed Summary: Git Integration for Azure Machine Learning**

---

## **Overview**

Azure Machine Learning (Azure ML) integrates with **Git** to track repository, branch, and commit information as part of training jobs. This allows users to maintain **version control**, collaborate on projects, and **reproduce experiments** effectively.

### **Key Features:**

- **Track source code** for training jobs in Azure ML.
- **Clone Git repositories** into the Azure ML workspace.
- **Use SSH authentication** for secure access to private repositories.
- **Automatically log Git metadata** (repository URL, branch, commit hash) for reproducibility.
- **View Git tracking details** in the **Azure portal, CLI, or Python SDK**.

---

## **1. Using Git in an Azure ML Workspace**

Azure ML provides a **shared workspace file system** where multiple users can collaborate. Users can **clone, manage, and use Git repositories** from:

1. **A local machine** before submitting a training job.
2. **A shared file system** using a **compute instance**.
3. **CI/CD pipelines** for automated model deployment.

### **Best Practices for Cloning Repositories**

- Clone repos **inside user-specific directories** (avoids conflicts with other users).
- **Local cloning** (inside the compute instance) provides better **performance**.
- **Shared file system cloning** ensures persistence, even if the compute instance is deleted.

---

## **2. Cloning a Git Repository via SSH**

Users can **securely authenticate** their Git repositories via **SSH keys**.

### **Step 1: Generate an SSH Key**

Run the following command on an **Azure ML compute instance terminal**:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

- The system will prompt for a **file location**. Press **Enter** to save it in the default location (`/home/azureuser/.ssh/`).
- Set a **passphrase** for additional security.

### **Step 2: Add the Public SSH Key to Git**

Retrieve the public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

- Copy the **output** and add it to your Git service:
  - **GitHub**: [Add SSH key to GitHub](https://github.com/settings/keys)
  - **GitLab**: [Add SSH key to GitLab](https://gitlab.com/profile/keys)
  - **Azure DevOps**: [Add SSH key to Azure DevOps](https://dev.azure.com/)
  - **Bitbucket**: [Add SSH key to Bitbucket](https://bitbucket.org/account/settings/)

### **Step 3: Clone a Repository via SSH**

```bash
git clone git@example.com:GitUser/azureml-example.git
```

- The system may prompt to **verify the SSH fingerprint** for security.
- If prompted, **enter the passphrase** for the SSH key.
- The repo is now cloned and can be used inside Azure ML.

---

## **3. Tracking Git Information in Azure ML**

When submitting a **training job**, Azure ML **automatically tracks Git metadata** if:

- The **source files** are inside a **Git repository**.
- The **git command-line tool** is available on the system.

### **Git Properties Automatically Logged**

| **Property**                 | **Git Command**                 | **Description**                                            |
| ---------------------------- | ------------------------------- | ---------------------------------------------------------- |
| `azureml.git.repository_uri` | `git ls-remote --get-url`       | Repository URL                                             |
| `azureml.git.branch`         | `git symbolic-ref --short HEAD` | Active branch when job was submitted                       |
| `azureml.git.commit`         | `git rev-parse HEAD`            | Commit hash of submitted job                               |
| `azureml.git.dirty`          | `git status --porcelain .`      | `True` if there are uncommitted changes, `False` otherwise |

### **How to Check if Git is Available**

Run:

```bash
git --version
```

- If installed, the output will show a version number like: `git version 2.43.0`
- If not installed, follow [Git installation instructions](https://git-scm.com/downloads).

---

## **4. Viewing Git Information in Azure ML**

Users can **retrieve Git metadata** using the **Azure portal, Python SDK, or CLI**.

### **4.1 View Git Information in Azure ML Studio (Portal)**

1. **Go to**: Azure ML Studio → **Jobs**.
2. **Select** a training job.
3. In the **Properties** section, select **Raw JSON**.
4. Look for Git-related properties:

```json
"properties": {
    "azureml.git.repository_uri": "git@github.com:azure/example-repo",
    "azureml.git.branch": "main",
    "azureml.git.commit": "d2a4c3f4b1e...",
    "azureml.git.dirty": "False"
}
```

### **4.2 Retrieve Git Information via Python SDK**

After submitting a training job, retrieve the **commit hash**:

```python
job = ml_client.jobs.get("my-job-id")
print(job.properties["azureml.git.commit"])
```

### **4.3 Retrieve Git Information via Azure CLI**

Run:

```bash
az ml job show --name my-job-id --query "{GitCommit:properties.azureml.git.commit}"
```

Output:

```json
{
  "GitCommit": "d2a4c3f4b1e..."
}
```

---

## **5. Best Practices for Git Integration in Azure ML**

### **✔ Version Control Best Practices**

- **Commit changes** before submitting a training job.
- **Use feature branches** for different experiments.
- **Use Git tags** to mark specific model versions.

### **✔ Managing Reproducibility**

- Always ensure **Git is tracking your experiments**.
- Use **commit hashes** to trace **exact code versions**.
- Use **MLflow** with Azure ML for additional experiment tracking.

### **✔ Automating Git Integration with CI/CD**

- Use **Azure DevOps Pipelines** or **GitHub Actions** to **automate model training & deployment**.
- Store **trained models in Azure ML Registry**.
- Automate **model retraining** when new commits are pushed.

---

## **6. Summary & Key Takeaways**

✔ **Seamless Git integration** allows version control and reproducibility in Azure ML.  
✔ **Clone repositories** directly into Azure ML using SSH authentication.  
✔ **Git metadata is automatically tracked** when submitting training jobs.  
✔ **Retrieve Git information** using the Azure ML **portal, Python SDK, or CLI**.  
✔ **Follow best practices** to maintain clean and reproducible workflows.

---

## **7. Next Steps**

📌 **Learn More About Git & Azure ML**:

- [Using Git with Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/)
- [MLflow & Azure ML Tracking](https://docs.microsoft.com/azure/machine-learning/how-to-use-mlflow)
- [Deploy ML Models with CI/CD](https://docs.microsoft.com/azure/machine-learning/how-to-cicd)

🚀 **Enhance Your MLOps Workflows with Git Version Control!**
