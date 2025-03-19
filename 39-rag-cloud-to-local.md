### **Detailed Summary of "Work with RAG Prompt Flows Locally (Preview)"**

#### **Overview**

This article explains how to transition **Retrieval-Augmented Generation (RAG) workflows** from **Azure Machine Learning (AML) cloud workspace** to a **local development environment**. Using the **Prompt flow extension in Visual Studio Code (VS Code)**, users can modify, test, and run **RAG prompt flows** locally.

> **Note:**  
> This feature is in **public preview** and is **not recommended for production workloads** due to possible limitations.

---

### **Prerequisites**

Before working with **RAG prompt flows locally**, ensure you have:

1. **Python Installed Locally**.
2. Install the required Python packages:
   ```sh
   pip install promptflow promptflow-tools
   ```
3. Install the **Promptflow Vector Database (vectordb) tool**:
   ```sh
   pip install promptflow-vectordb
   ```
4. **Visual Studio Code (VS Code)** with:
   - **Python extension**.
   - **Prompt flow extension**.
5. **An Azure OpenAI Account** with model deployments for:
   - **Chat models**.
   - **Text embeddings**.
6. **A Vector Index** created in **Azure Machine Learning Studio**.

---

### **Steps to Create and Download a Prompt Flow**

#### **Step 1: Create a Prompt Flow in Azure Machine Learning**

1. **Set up connections**:
   - In **AML Studio**, go to the **Connections tab**.
   - Connect to your **Azure OpenAI resource**.
   - If using **Azure AI Search**, ensure a connection is established.
2. **Create the RAG Prompt Flow**:

   - Navigate to **Prompt Flow** in AML Studio.
   - Click **Create**.
   - Select **Clone** on the **Q&A on Your Data** sample.

3. **Configure the Prompt Flow**:

   - In the **lookup step**, add your **vector index information**.
   - In the **answer_the_question_with_context step**, provide:
     - **Connection and Deployment details** for **chat API**.

4. **Run and Test the Flow**:
   - Verify the flow **executes correctly**.
   - Click **Save**.

#### **Step 2: Download the Prompt Flow to Your Local Machine**

1. Click the **Download icon** in the **Files section** of the AML Studio.
2. Extract the **ZIP package** to a folder on your local device.

---

### **Working with the Flow in VS Code**

After downloading the prompt flow, users can modify and test it locally using **VS Code**.

#### **Step 1: Open the Prompt Flow in VS Code**

1. Open **VS Code** and navigate to the **unpacked prompt flow folder**.
2. Click the **Prompt Flow icon** in the left menu.
3. Install dependencies:
   - Select **Install Dependencies** in the **management pane**.
   - Ensure the correct **Python interpreter** is selected.
   - Confirm that **promptflow** and **promptflow-tools** are installed.

#### **Step 2: Create Required Connections**

To use the **vector index lookup tool locally**, establish the same **connections** used in **AML Studio**.

1. Expand the **Connections** section.
2. Click the **+ icon** next to **AzureOpenAI**.
3. A new YAML file (`new_AzureOpenAI_connection.yaml`) opens.
4. **Edit the file** to include:
   - **Azure OpenAI connection name**.
   - **API Base or Endpoint** (DO NOT enter the API key here).
5. Click **Create Connection** and enter the **API key** in the **terminal**.

> **Note:** If using **Azure AI Search** as a **vector data source**, create an additional connection for **vector index lookup**.

---

### **Checking and Editing Files**

#### **Step 1: Open the Flow Configuration File**

1. In **VS Code**, open **flow.dag.yaml**.
2. Click **Visual Editor**.

#### **Step 2: Validate Vector Index Configuration**

1. Scroll to the **lookup node**.
2. Ensure the **mlindex_content paths and connections** are correctly set.

> **Note:** If the **indexed docs** are an **Azure data asset**, local access requires **Azure authentication**.

#### **Step 3: Update the Lookup Node**

1. Select the **Edit icon** in the queries input box.
2. Locate the **lookup node definition** in `flow.dag.yaml`.
3. Ensure the **tool section** is set to:
   ```yaml
   tool: promptflow_vectordb.tool.vector_index_lookup.VectorIndexLookup.search
   ```
   - This is the **local version** of the **vector index lookup tool**.

#### **Step 4: Verify the Python Code**

1. Scroll to the **generate_prompt_context node**.
2. Click **Open Code File**.
3. Ensure the **package name** is set to:
   ```python
   promptflow_vectordb
   ```

#### **Step 5: Validate Answer Node Configuration**

1. Navigate to **answer_the_question_with_context**.
2. Ensure it references the **local connection**.
3. Verify the **deployment_name** matches the embedding model.

---

### **Testing and Running the Local Prompt Flow**

1. At the **top of the flow**, enter an **input question** (e.g., `How to use SDK V2?`).
2. Click the **Run icon** to execute the flow.

---

### **Next Steps**

- **Get started with Prompt Flow**.
- **Create a Vector Index in Azure Machine Learning**.
- **Use Index Lookup Tool for Azure AI Foundry**.
- **Integrate Prompt Flow with LLM-based DevOps**.

---

### **Conclusion**

This article provides a **step-by-step guide** on transitioning **RAG prompt flows** from **Azure Machine Learning to a local development environment**.  
By using **VS Code** and the **Prompt Flow extension**, users can:

- **Edit, test, and run** RAG workflows locally.
- **Modify prompt engineering settings** for better response accuracy.
- **Work offline** while maintaining integration with **Azure AI services**.

This approach **streamlines development** and **enhances control** over RAG implementations without relying on **cloud-only execution**.
