### **Detailed Summary of "Create a Vector Index in an Azure Machine Learning Prompt Flow (Preview)"**

#### **Overview**

This article explains how to create a **vector index** in **Azure Machine Learning (AML)** using **prompt flow**. A vector index allows AI models to efficiently retrieve relevant data by converting information into numerical embeddings. Users can create a vector index from multiple sources such as:

- **Local files/folders**
- **Cloud storage**
- **Azure Machine Learning data assets**
- **Git repositories**
- **SQL databases**

**Supported File Types:**  
AML can process **.txt, .md, .pdf, .xls, .docx**, and other document formats to create embeddings.

Users can either create a **new vector index** or **reuse an existing Azure AI Search index**.

---

### **How Azure Machine Learning Processes Vector Indexes**

When a vector index is created, AML performs the following actions:

1. **Chunks the data** into smaller, retrievable sections.
2. **Generates embeddings** (numerical representations of the text).
3. **Stores embeddings** in either a **Faiss index** or an **Azure AI Search index**.

In addition, AML provides:

- **Test data generation** for the selected data source.
- **A sample prompt flow**, which includes:
  - **Automatically generated prompt variants**.
  - **Evaluation metrics** to compare different prompt versions.
  - **Performance analysis** to choose the best-performing variant.

---

### **Prerequisites**

Before creating a vector index, users must have:

1. **An Azure Subscription**: Users without an account can create a **free Azure account**.
2. **Access to Azure OpenAI Service**.
3. **Enabled Prompt Flows in AML Workspace**:
   - Navigate to **Manage Preview Features**.
   - Enable **Build AI Solutions with Prompt Flow**.

---

### **Steps to Create a Vector Index in Machine Learning Studio**

#### **Step 1: Access the Vector Index Tab**

1. In **Azure Machine Learning Studio**, select **Prompt Flow** from the left menu.
2. Click the **Vector Index** tab.
3. Select **Create**.

#### **Step 2: Configure the Vector Index**

1. Enter a **name** for the vector index.
2. Choose a **data source type** (e.g., local file, cloud storage, Git repository).
3. Provide the **data source location** and click **Next**.

#### **Step 3: Review and Create**

1. Review the configured details.
2. Click **Create**.
3. Once created, track the **status** of the vector index on the **overview page**.
4. The processing time depends on the **size of the dataset**.

---

### **Adding a Vector Index to a Prompt Flow**

Once the vector index is created, it can be linked to a **prompt flow** for AI-enhanced search capabilities.

#### **Step 1: Open an Existing Prompt Flow**

1. In **Azure Machine Learning Studio**, open a previously created **prompt flow**.

#### **Step 2: Add the Vector Index**

1. In the **prompt flow designer**, select **More Tools**.
2. Choose **Index Lookup** from the list.

#### **Step 3: Configure the Vector Index**

1. Enter a **name** for the vector index.
2. Select the **mlindex_content** value box and choose the created vector index.
3. Click **Save**.

#### **Step 4: Define Queries**

1. Enter **queries and query types** to be performed against the index.
   - Example query: `"How to use SDK V2?"`
   - Example embedding: `${embed_the_question.output}`
2. Queries can be in **plain text** or **embedding format**.

---

### **Supported File Types**

Users can create a vector index from the following file types:

- **Text Files**: `.txt`, `.md`, `.html`, `.htm`, `.py`
- **Documents**: `.pdf`, `.ppt`, `.pptx`, `.doc`, `.docx`
- **Spreadsheets**: `.xls`, `.xlsx`

**Note:** Unsupported file types will be ignored.

---

### **Next Steps**

After creating a vector index, users can:

1. **Get started with RAG using a prompt flow sample** (preview).
2. **Use vector stores with Azure Machine Learning** (preview).

---

### **Conclusion**

This guide provides a step-by-step process to **create, manage, and integrate vector indexes** within **Azure Machine Learning prompt flows**. Vector indexes enable efficient **retrieval-augmented generation (RAG)** workflows, helping **LLMs retrieve and process relevant data** efficiently. Users can choose between **Faiss for local use** and **Azure AI Search for enterprise-scale search capabilities**.
