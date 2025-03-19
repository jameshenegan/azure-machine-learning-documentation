### **Detailed Summary of "Secure Your RAG Workflows with Network Isolation (Preview)"**

#### **Overview**

This article provides guidance on how to **secure Retrieval-Augmented Generation (RAG) workflows** in **Azure Machine Learning (AML)** using **network isolation**. There are two main **network management options** for securing RAG workflows:

1. **Managed Virtual Network (Managed VNet)** – Azure’s built-in networking solution.
2. **Bring Your Own (BYO) Custom Virtual Network (VNet)** – Gives full control over network settings, including **subnets, firewalls, and security rules**.

For the **Managed VNet** option, users can choose between:

- **Allow Internet Outbound**: Permits outbound internet traffic.
- **Allow Only Approved Outbound**: Restricts outbound traffic to approved destinations.

Depending on the chosen configuration, **additional steps** may be required to ensure complete **network isolation**.

> **Note:**  
> This feature is in **public preview**, meaning some features may be **limited or unsupported**, and it's **not recommended for production workloads**.

---

### **Prerequisites**

To secure an **RAG workflow**, users need:

1. **An Azure Subscription** (a **free account** can be created if unavailable).
2. **Access to Azure OpenAI Service**.
3. **A Secure AML Workspace**, configured with:
   - **Managed Virtual Network** or
   - **Custom Virtual Network** (BYO VNet).
4. **Prompt Flows Enabled in AML**:
   - Navigate to **Manage Preview Features**.
   - Enable **Build AI Solutions with Prompt Flow**.

---

### **Securing RAG with Azure Machine Learning Managed VNet**

#### **Step 1: Enable Managed VNet**

1. In the **Azure Portal**, go to **Networking** under **Settings** in the AML workspace.
2. Follow the **Workspace Managed Network Isolation** guide to enable a **Managed VNet**.

#### **Step 2: Configure Network Rules for Private Cognitive Services**

1. To allow **RAG workflows** to communicate with **Azure Cognitive Services** (e.g., **Azure OpenAI, Azure AI Search**):
   - In **Networking Settings**, select **Workspace Managed Outbound Access**.
   - Click **+Add User-Defined Outbound Rule**.
   - Enter a **Rule Name** and select the **Azure resource** (e.g., **OpenAI or AI Search**).
2. **AML automatically creates a private endpoint** for the selected resource.
3. If the endpoint status is **pending**, manually approve it in the **Azure Portal**.

#### **Step 3: Grant Storage Access to AML Managed Identity**

1. Navigate to the **storage account** linked to the workspace.
2. Under **Access Control (IAM)**, click **Add Role Assignment**.
3. Assign the following roles to the **Workspace Managed Identity**:
   - **Storage Table Data Contributor**
   - **Storage Blob Data Contributor**
4. Ensure the **Managed Identity option is selected**, then choose **Azure Machine Learning Workspace**.

#### **Step 4 (Optional): Configure Outgoing FQDN Rules**

1. If using **Allow Only Approved Outbound** for **Managed VNet**, configure an **FQDN rule**:
   - In **Networking Settings**, click **+Add User-Defined Outbound Rule**.
   - Select **FQDN Rule** as the **Destination Type**.
   - Enter the **endpoint URL** of your **Azure OpenAI service**.
2. Without this rule, even **public OpenAI resources won’t be accessible** for **embedding operations**.

#### **Step 5 (Optional): Configure Private Storage for Secure Data Upload**

1. To upload data securely, ensure that **Azure Machine Learning** can access the storage account.
2. In **Storage Account Networking Settings**:
   - Enable **Selected Virtual Network and IPs**.
   - Add the **AML workspace subnet**.

---

### **Securing RAG with Bring Your Own (BYO) Custom VNet**

1. **Select "Use My Own Virtual Network"** when setting up the AML workspace.
2. **Manually configure network rules**:
   - Set up **private endpoints** for related services.
   - Define **firewall rules** and **Network Security Group (NSG) settings**.
3. **Ensure compute compatibility**:
   - In **Vector Index Creation Wizard**, select **Compute Instance or Compute Cluster**.
   - **Serverless Compute is not supported** in this scenario.

---

### **Troubleshooting Common Network Issues**

#### **Issue: Compute Fails to Start**

- **Fix**: Add a **placeholder FQDN rule** in **Networking Settings** to trigger a **managed network update**, then **recreate the compute instance**.

#### **Issue: Resource Not Registered with Microsoft.Network**

- **Fix**: Ensure that the **Azure subscription is registered** with the **Microsoft.Network resource provider**:
  - Go to **Subscription > Resource Providers**.
  - Verify that **Microsoft.Network** is registered.

#### **Issue: Serverless Job Queued for 10-15 Minutes**

- **Fix**: **Managed Network provisioning** for **private endpoints** can take time. This delay is **expected only for the first-time setup**.

---

### **Next Steps**

- **Secure Your Prompt Flow**
- **Further Network Security Configurations**

---

### **Conclusion**

Azure Machine Learning provides **network isolation** options to **secure RAG workflows** using **Managed Virtual Networks (VNet)** or **Bring Your Own Custom VNet**.

- **Managed VNet** simplifies security by offering **pre-configured isolation**, while
- **BYO VNet** provides **full control over networking rules**.

To ensure secure operations, users must configure:

- **Private network rules for Azure Cognitive Services**
- **Storage access permissions for AML Managed Identity**
- **FQDN rules for outbound connections**
- **Private endpoints for custom networking scenarios**

With **proper network isolation**, **RAG workflows remain secure**, protecting sensitive **LLM interactions and data storage**.
