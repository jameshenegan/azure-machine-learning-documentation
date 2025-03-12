# Quickstart: Get started with Azure Machine Learning

https://learn.microsoft.com/en-us/azure/machine-learning/tutorial-azure-ml-in-a-day

### **Detailed Summary: Quickstart - Get Started with Azure Machine Learning**

This **quickstart guide** introduces key **Azure Machine Learning (Azure ML) concepts** by walking through the process of **training, registering, deploying, and testing a machine learning model**. The tutorial covers setting up a workspace, running a training job on a compute cluster, and deploying the trained model as an endpoint.

---

## **Key Steps in the Quickstart**

### **1. Prerequisites**

Before starting, ensure you have:

- An **Azure Machine Learning workspace**.
- A **compute instance** (cloud-based development environment).
- **Python SDK (`azure-ai-ml` v2)** installed.

### **2. Set Up the Development Environment**

- Open **Azure Machine Learning Studio** and select your workspace.
- Create or open a **Jupyter Notebook** inside the studio.
- Set the **Python kernel** to **3.10 - SDK v2**.
- If needed, **authenticate** using Azure credentials.
- Open the notebook in **Visual Studio Code** for an integrated development experience.

---

## **3. Connect to Azure ML Workspace**

A workspace is the **central resource** for ML experiments. To reference it in your script:

1. **Initialize MLClient**

   ```python
   from azure.ai.ml import MLClient
   from azure.identity import DefaultAzureCredential

   credential = DefaultAzureCredential()
   ml_client = MLClient(
       credential=credential,
       subscription_id="<SUBSCRIPTION_ID>",
       resource_group_name="<RESOURCE_GROUP>",
       workspace_name="<AML_WORKSPACE_NAME>",
   )
   ```

2. **Verify Workspace Connection**
   ```python
   ws = ml_client.workspaces.get("<AML_WORKSPACE_NAME>")
   print(ws.location, ":", ws.resource_group)
   ```

---

## **4. Create a Training Script**

A **training script** (`main.py`) is needed for preprocessing data, training a model, and saving it.

### **Steps:**

1. **Create a source directory**
   ```python
   import os
   train_src_dir = "./src"
   os.makedirs(train_src_dir, exist_ok=True)
   ```
2. **Write the Training Script (`main.py`)**

   ```python
   %%writefile {train_src_dir}/main.py
   import os
   import argparse
   import pandas as pd
   import mlflow
   from sklearn.ensemble import GradientBoostingClassifier
   from sklearn.model_selection import train_test_split
   from sklearn.metrics import classification_report

   def main():
       parser = argparse.ArgumentParser()
       parser.add_argument("--data", type=str, help="path to input data")
       parser.add_argument("--test_train_ratio", type=float, default=0.25)
       parser.add_argument("--n_estimators", type=int, default=100)
       parser.add_argument("--learning_rate", type=float, default=0.1)
       parser.add_argument("--registered_model_name", type=str, help="model name")
       args = parser.parse_args()

       mlflow.start_run()
       mlflow.sklearn.autolog()

       credit_df = pd.read_csv(args.data, header=1, index_col=0)
       train_df, test_df = train_test_split(credit_df, test_size=args.test_train_ratio)

       y_train = train_df.pop("default payment next month")
       X_train = train_df.values
       y_test = test_df.pop("default payment next month")
       X_test = test_df.values

       clf = GradientBoostingClassifier(n_estimators=args.n_estimators, learning_rate=args.learning_rate)
       clf.fit(X_train, y_train)
       y_pred = clf.predict(X_test)

       print(classification_report(y_test, y_pred))

       # Register Model
       mlflow.sklearn.log_model(clf, registered_model_name=args.registered_model_name, artifact_path=args.registered_model_name)
       mlflow.end_run()

   if __name__ == "__main__":
       main()
   ```

---

## **5. Configure and Submit the Training Job**

### **Define Training Job Configuration**

- Specifies the **input dataset**.
- Defines the **compute resource** (serverless compute is used by default).
- Uses a **predefined environment** with necessary dependencies.

```python
from azure.ai.ml import command
from azure.ai.ml import Input

registered_model_name = "credit_defaults_model"

job = command(
    inputs=dict(
        data=Input(
            type="uri_file",
            path="https://azuremlexamples.blob.core.windows.net/datasets/credit_card/default_of_credit_card_clients.csv",
        ),
        test_train_ratio=0.2,
        learning_rate=0.25,
        registered_model_name=registered_model_name,
    ),
    code="./src/",
    command="python main.py --data ${{inputs.data}} --test_train_ratio ${{inputs.test_train_ratio}} --learning_rate ${{inputs.learning_rate}} --registered_model_name ${{inputs.registered_model_name}}",
    environment="azureml://registries/azureml/environments/sklearn-1.5/labels/latest",
    display_name="credit_default_prediction",
)
```

### **Submit the Job**

```python
ml_client.create_or_update(job)
```

- The job runs **training, evaluation, and model registration**.
- The process takes **a few minutes** depending on compute resources.

---

## **6. Deploy the Model as an Online Endpoint**

### **Create a Unique Endpoint Name**

```python
import uuid
online_endpoint_name = "credit-endpoint-" + str(uuid.uuid4())[:8]
```

### **Deploy the Model**

1. **Create an Online Endpoint**

   ```python
   from azure.ai.ml.entities import ManagedOnlineEndpoint

   endpoint = ManagedOnlineEndpoint(
       name=online_endpoint_name,
       description="Online endpoint for credit default prediction",
       auth_mode="key",
   )

   endpoint = ml_client.online_endpoints.begin_create_or_update(endpoint).result()
   print(f"Endpoint {endpoint.name} provisioning state: {endpoint.provisioning_state}")
   ```

2. **Deploy the Model to the Endpoint**

   ```python
   from azure.ai.ml.entities import ManagedOnlineDeployment, Model

   latest_model_version = max([int(m.version) for m in ml_client.models.list(name=registered_model_name)])

   model = ml_client.models.get(name=registered_model_name, version=latest_model_version)

   blue_deployment = ManagedOnlineDeployment(
       name="blue",
       endpoint_name=online_endpoint_name,
       model=model,
       instance_type="Standard_DS3_v2",
       instance_count=1,
   )

   blue_deployment = ml_client.begin_create_or_update(blue_deployment).result()
   ```

   - Deployment takes **6-8 minutes**.
   - The model is now available as a **REST API**.

---

## **7. Test the Deployed Model**

### **Create a Sample Input File**

```python
deploy_dir = "./deploy"
os.makedirs(deploy_dir, exist_ok=True)

%%writefile {deploy_dir}/sample-request.json
{
  "input_data": {
    "columns": [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22],
    "index": [0, 1],
    "data": [
            [20000,2,2,1,24,2,2,-1,-1,-2,-2,3913,3102,689,0,0,0,0,689,0,0,0,0],
            [10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 10, 9, 8]
        ]
  }
}
```

### **Invoke the Endpoint**

```python
ml_client.online_endpoints.invoke(
    endpoint_name=online_endpoint_name,
    request_file="./deploy/sample-request.json",
    deployment_name="blue",
)
```

---

## **8. Clean Up Resources**

### **Delete Endpoint**

```python
ml_client.online_endpoints.begin_delete(name=online_endpoint_name)
```

### **Stop Compute Instance**

- Navigate to **Compute > Compute Instances** in Azure ML Studio.
- Select the instance and click **Stop**.

### **Delete Azure Resources**

- Go to the **Azure Portal > Resource Groups**.
- Select your **resource group** and click **Delete**.

---

## **Next Steps**

- **Learn about data storage** in Azure ML.
- **Explore cloud-based model development**.
- **Train and deploy more complex models**.

This quickstart covers the **entire lifecycle** of an ML model in **Azure Machine Learning**, making it easier to scale **training, deployment, and inference** workflows efficiently. 🚀
