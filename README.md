# Fabric Model → Azure ML Endpoint (via MLFlow)

Build a model in Microsoft Fabric, push it to Azure ML, and deploy it as a managed endpoint — all from a single notebook.

Fabric doesn't natively support deploying models to Azure ML, so we register an Entra ID app and use its credentials to push the model via MLFlow from the Fabric notebook into an Azure ML workspace.

## Prerequisites

- A Microsoft Fabric workspace
- An Azure ML workspace
- The notebook: [`Time series-413.ipynb`](Time%20series-413.ipynb) — import this into your Fabric workspace

This is a standard timeseries model example. **Step 1, Step 6, and Step 7** in the notebook are the relevant parts for the Fabric → Azure ML integration.

---

## 1. Register an App in Entra ID

Go to [entra.microsoft.com](https://entra.microsoft.com/).

![Entra portal](img/entra_front_1.png)

In the left sidebar, click **App Registrations** → **New registration**.

![App registrations](img/entra_app_reg_2.png)

Give it a name and click **Register**.

![Register app](img/entra_app_reg_detail_3.png)

## 2. Save App Credentials

Copy the **Application (client) ID** and **Directory (tenant) ID** — you'll need these.

![Copy IDs](img/entra_app_dir_id_4.png)

In the left sidebar of the app registration, go to **Certificates & secrets** → **New client secret** → **Add** (defaults are fine).

![Generate secret](img/entra_client_secret_gen_5.png)

Copy the **Value** from your secret. This is your `client_secret` — save it.

![Copy secret value](img/entra_client_secret_value_6.png)

## 3. Configure the Fabric Notebook

In the Fabric notebook, go to **Step 1: Configurations** and find the first code block. Replace all the env values with the credentials you saved:

![Set config](img/fabric_run_config_7.png)

Run that cell. Once done, **delete it or comment it out** — you don't want credentials sitting in a cell.

![Comment out config](img/fabric_comment_out_conf_8.png)

## 4. Grant Azure ML Access to the App

Go to [portal.azure.com](https://portal.azure.com/) and navigate to your Azure ML resource.

![Azure ML resource](img/azure_azureml_9.png)

Go to **IAM** in the left pane → **Add** → **Add role assignment**.

![Add role assignment](img/azure_azureml_iam_add_10.png)

Search and select the role **AzureML Data Scientist**, click **Next**.

![Select role](img/azure_azureml_role_assign_11.png)

Click **Select members**, find your Entra app by name, click **Select** → **Review + Assign**.

![Assign to app](img/azure_azureml_entra_app_12.png)

## 5. Create a Custom Environment

Automatic environments don't work when pushing a model from Fabric → Azure ML, so we create a custom one.

In the Fabric notebook, right under the config cell, there's a cell for custom environment creation on Azure ML. **Run it**.

![Create environment](img/fabric_create_endpoint_13.png)

Once done, **comment it out** — you only need to create the environment once.

![Comment out env cell](img/fabric_comment_out_endpoint_14.png)

## 6. Run the Notebook

Go back to your Fabric workspace and **Run All**. The notebook will build the model, push it to Azure ML, and deploy it as a managed endpoint — all versioned automatically.

![Run all](img/fabric_run_all_15.png)

## 7. Verify in Azure ML

Go to [ml.azure.com](https://ml.azure.com/). In the left pane → **Models**. You'll see `my-prophet-model`. Multiple runs will auto-version.

![Models in Azure ML](img/azureml_models_16.png)

To test, go to **Endpoints** in the left pane and click on your endpoint.

![Endpoints](img/azureml_endpoints_17.png)

Go to the **Test** tab and use this input:

```json
{
  "input_data": {
    "columns": ["ds"],
    "data": [["2012-11-26"]]
  }
}
```

![Test endpoint](img/azureml_endpoint_test_18.png)

You'll see the predictions in the output. This endpoint can be called from anywhere — a Fabric ML pipeline developed and hosted as an Azure ML endpoint.

## Disclaimer

> [!WARNING]
> This repository is an **unofficial** guide. It is provided "as is" without warranty of any kind. Use at your own risk. The authors and contributors assume no liability for any damages or issues arising from its use. See the [LICENSE](LICENSE) for full terms.
