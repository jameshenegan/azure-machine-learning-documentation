# Tutorial: Train a model in Azure Machine Learning

https://learn.microsoft.com/en-us/azure/machine-learning/tutorial-train-model

### **Detailed Summary: Train a Model in Azure Machine Learning**

This tutorial guides data scientists through **training a machine learning model** in **Azure Machine Learning (Azure ML)** using a **command job**. The tutorial uses a **credit card dataset** to predict customers likely to **default on payments**. It covers data preparation, model training, and registering the trained model in the **Azure ML workspace**.

---

## **Key Steps in the Tutorial**

### **1. Prerequisites**

To follow this tutorial, ensure:

- You have an **Azure Machine Learning workspace**.
- You can **sign in to Azure ML Studio**.
- A **compute instance is available** (or create one).
- The **Python kernel is set to SDK v2 (Python 3.10)**.

⚠ **If your workspace is in a managed virtual network, configure outbound rules for access to Python package repositories.**

---

## **2. Setting Up the Development Environment**

- **Open or create a Jupyter Notebook** in Azure ML Studio.
- If needed, **clone the sample notebook** from:
  ```
  tutorials/get-started-notebooks/train-model.ipynb
  ```
- **Ensure the compute instance is running** before proceeding.
- Optionally, **open the notebook in VS Code** for an enhanced development experience.

---

## **3. Train a Model Using a Command Job**

Azure ML **command jobs** allow running **custom training scripts** in a specified environment. A **command job** includes:

- **A training script**
- **A compute resource**
- **An environment (dependencies)**
- **A command to execute the script**

This tutorial sets up a **command job** to train a **classifier** predicting credit card defaults.

---

## **4. Connect to Azure ML Workspace**

### **Initialize ML Client**

```python
from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential

# Authenticate
credential = DefaultAzureCredential()

# Connect to workspace
ml_client = MLClient(
    credential=credential,
    subscription_id="<SUBSCRIPTION_ID>",
    resource_group_name="<RESOURCE_GROUP>",
    workspace_name="<AML_WORKSPACE_NAME>",
)
```

### **Verify Connection**

```python
ws = ml_client.workspaces.get("<AML_WORKSPACE_NAME>")
print(ws.location, ":", ws.resource_group)
```

---

## **5. Create a Job Environment**

Azure ML **environments** define the **software runtime** for ML jobs. You can use **predefined environments** or **create a custom one**.

### **Create a Conda Environment File**

```python
import os

dependencies_dir = "./dependencies"
os.makedirs(dependencies_dir, exist_ok=True)

# Create a conda.yaml file
%%writefile {dependencies_dir}/conda.yaml
name: model-env
channels:
  - conda-forge
dependencies:
  - python=3.8
  - numpy=1.21.2
  - pip=21.2.4
  - scikit-learn=1.0.2
  - scipy=1.7.1
  - pandas>=1.1,<1.2
  - pip:
    - inference-schema[numpy-support]==1.3.0
    - mlflow==2.8.0
    - mlflow-skinny==2.8.0
    - azureml-mlflow==1.51.0
    - psutil>=5.8,<5.9
    - tqdm>=4.59,<4.60
    - ipykernel~=6.0
    - matplotlib
```

### **Register the Custom Environment in Azure ML**

```python
from azure.ai.ml.entities import Environment

custom_env_name = "aml-scikit-learn"

custom_job_env = Environment(
    name=custom_env_name,
    description="Custom environment for Credit Card Defaults job",
    tags={"scikit-learn": "1.0.2"},
    conda_file=os.path.join(dependencies_dir, "conda.yaml"),
    image="mcr.microsoft.com/azureml/openmpi4.1.0-ubuntu20.04:latest",
)

custom_job_env = ml_client.environments.create_or_update(custom_job_env)

print(f"Environment {custom_job_env.name} registered with version {custom_job_env.version}")
```

---

## **6. Create a Training Script**

### **Set Up Training Script Directory**

```python
import os

train_src_dir = "./src"
os.makedirs(train_src_dir, exist_ok=True)
```

### **Write the Training Script (`main.py`)**

This script:

- Prepares **data**.
- Trains a **Gradient Boosting Classifier**.
- Registers the trained **model** in Azure ML.

```python
%%writefile {train_src_dir}/main.py
import os
import argparse
import pandas as pd
import mlflow
import mlflow.sklearn
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.metrics import classification_report
from sklearn.model_selection import train_test_split

def main():
    """Main function of the script."""
    parser = argparse.ArgumentParser()
    parser.add_argument("--data", type=str, help="path to input data")
    parser.add_argument("--test_train_ratio", type=float, default=0.25)
    parser.add_argument("--n_estimators", type=int, default=100)
    parser.add_argument("--learning_rate", type=float, default=0.1)
    parser.add_argument("--registered_model_name", type=str, help="model name")
    args = parser.parse_args()

    mlflow.start_run()
    mlflow.sklearn.autolog()

    # Load Data
    credit_df = pd.read_csv(args.data, header=1, index_col=0)
    train_df, test_df = train_test_split(credit_df, test_size=args.test_train_ratio)

    # Extract Features and Labels
    y_train, X_train = train_df.pop("default payment next month"), train_df.values
    y_test, X_test = test_df.pop("default payment next month"), test_df.values

    # Train Model
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

## **7. Configure the Command Job**

A **command job** is created to run the training script in the registered **environment**.

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
    environment="aml-scikit-learn@latest",
    display_name="credit_default_prediction",
)
```

---

## **8. Submit and Monitor the Job**

### **Submit the Training Job**

```python
ml_client.create_or_update(job)
```

The job takes **2-10 minutes** to complete.

### **Monitor Job Output**

- **Follow the job link in the output**.
- View metrics such as **training accuracy, loss, and logs** in Azure ML Studio.

---

## **9. Clean Up Resources**

### **Stop Compute Instance**

- Open **Azure ML Studio**.
- Navigate to **Compute > Compute Instances**.
- Select the instance and click **Stop**.

### **Delete Azure Resources**

- Go to **Azure Portal > Resource Groups**.
- Select your **resource group**.
- Click **Delete**.

---

## **Next Steps**

- **Deploy the trained model**: [Deploy a Model as an Endpoint](https://learn.microsoft.com/en-us/azure/machine-learning/tutorial-deploy-model)
- **Explore automated machine learning (AutoML)**.
- **Check out additional sample notebooks**.

---

## **Conclusion**

This tutorial provides a **step-by-step guide** for:
✅ **Setting up an Azure ML environment**.  
✅ **Training a classifier with a command job**.  
✅ **Registering the model in Azure ML**.  
✅ **Running and monitoring cloud-based training**.

This approach helps **streamline ML workflows**, enabling scalable and reproducible **model training** in Azure ML. 🚀
