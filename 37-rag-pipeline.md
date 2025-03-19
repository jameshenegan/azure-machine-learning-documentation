### **Detailed Summary of "Use Azure Machine Learning Pipelines with No Code to Construct RAG Pipelines (Preview)"**

#### **Overview**

This article explains how to build **Retrieval-Augmented Generation (RAG) pipelines** in **Azure Machine Learning (AML)** without writing extensive code. Users can leverage **built-in pipeline components** to handle various tasks such as:

- **Data chunking**
- **Embeddings generation**
- **Test data creation**
- **Automatic prompt generation**
- **Prompt evaluation**

For **advanced scenarios**, users can create custom **AML pipelines** using **notebooks**, allowing for granular control over the **RAG workflow**. Additionally, **vector indexes created in AML** can be integrated into **LangChain**.

> **Note:**  
> This feature is in **public preview**, meaning it may have **limited support and functionality** and is **not recommended for production workloads**.

---

### **Prerequisites**

Before constructing a RAG pipeline, users need:

1. **An Azure Subscription**: If unavailable, a **free Azure account** can be created.
2. **Access to Azure OpenAI Service**.
3. **Enable Prompt Flow in AML Workspace**:
   - Navigate to **Manage Preview Features**.
   - Enable **Build AI Solutions with Prompt Flow**.

---

### **Prompt Flow Pipeline Notebook Sample Repository**

Azure Machine Learning offers **notebook tutorials** to help users create and manage **RAG pipelines**.

#### **1. QA Data Generation**

- Helps identify the **best prompt** for RAG.
- Generates **evaluation metrics** for RAG performance.
- Demonstrates how to **create a QA dataset** from a **Git repository**.

#### **2. Test Data Generation and Auto Prompt**

- Uses **vector indexes** to build an **RAG model**.
- Evaluates **prompt flow performance** on a **test dataset**.

#### **3. Create a FAISS-Based Vector Index**

- Sets up an **AML pipeline** to:
  1. Pull data from a **Git repository**.
  2. Process and **chunk the data**.
  3. Generate **embeddings**.
  4. Create a **LangChain-compatible FAISS Vector Index**.

These notebooks allow users to **quickly implement RAG pipelines** in Azure Machine Learning.

---

### **Next Steps**

After setting up a RAG pipeline, users can explore:

1. **Creating a Vector Index in AML Prompt Flow**.
2. **Using Vector Stores with Azure Machine Learning**.

---

### **Conclusion**

Azure Machine Learning enables **no-code and low-code** RAG pipeline creation using **built-in pipeline components and notebooks**. These pipelines streamline **data preparation, vector indexing, and prompt evaluation**, making it easier to **integrate RAG into AI workflows**. For more customization, users can also **write their own AML pipelines** using **notebooks** and **LangChain integration**.
