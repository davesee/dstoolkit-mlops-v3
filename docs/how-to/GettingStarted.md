# Getting started with Model Factory

This solution supports Azure Machine Learning (ML) as a platform for ML, and Azure DevOps or github as a platform for operationalization. MLOps with Model Factory provides automation of the following:

- Infrastructure provisioning using either Azure Pipelines or github workflows using Terraform as the IaC language.
- A PR build triggered upon changes to one or more models.
- A CI build and deployment of one or more models to batch and online endpoints.

## Assumptions

- The user of this guide understands basic operations on Azure DevOps, github.com, visual studio code, or an IDE of their choice. Use the following guide to familiarize yourself with github [Getting started with your GitHub account](https://docs.github.com/en/get-started/onboarding/getting-started-with-your-github-account). Use the following guide to familiarize yourself with visual studio code [Visual Studio Code documentation](https://code.visualstudio.com/docs)

- Your team has an Azure Subscription within which to host Model Factory. If you don't have an Azure subscription, create a free account by following this link. [Free Azure Subscription](https://azure.microsoft.com/en-us/free/search/?ef_id=_k_67e7bdd2a501151df8d8d83b02edc75b_k_&OCID=AIDcmm5edswduu_SEM__k_67e7bdd2a501151df8d8d83b02edc75b_k_&msclkid=67e7bdd2a501151df8d8d83b02edc75b)

- You have created an app registration to be used to operate the infrastructure, and AML build and ci pipelines.

- You have granted the service principal above, at least Contributor, and User Access Administrator on the target subscription in Azure.
  \*\*Use this document as a reference when creating an app registration: [Create a Microsoft Entra application and service principal that can access resources](https://learn.microsoft.com/en-us/entra/identity-platform/howto-create-service-principal-portal)

## Setup your source control environment

**Step 1.** Clone the repository, create a _development_ branch, and make it the default branch so that all PRs merge to it. This guide assumes that the team works with a _development_ branch as the primary source for coding and improving model quality. Later, you can implement an Azure Pipeline to move code from the _development_ branch to qa/main or that executes a release process with each check-in. However, release management is not in scope of this guide.

## Azure DevOps Setup

**Step 1.** Delete the .github directory.

**Step 2.** Create an Azure DevOps organization and project. Follow the instructions here: [Create a project in Azure DevOps](https://learn.microsoft.com/en-us/azure/devops/organizations/projects/create-project?view=azure-devops&tabs=browser)

**Step 3.** In Azure Pipelines > Library, create a new variable group named **"mlops_platform_dev_vg"**, add the variables and their values listed below: (Information about variable groups in Azure DevOps can be found in [Add & use variable groups](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops&tabs=classic).

**Note To provision test or production infrastructure create a new variable group, add the required variables, and modify the reference to the variable group in infra_provision_terraform_pipeline.yml files.**)

> ⚠️ Some azure resource names have to be unique within your Azure subscription or region. Please make sure to use unique names. One strategy is to append a three-part version to the names defined in the variables below (ie. For AML workspace, you might use "aml-mlw-001").

**Mandatory Infrastructure variables for terraform provisioning.**

- "APPINSIGHTS_NAME": Set to a value of your choosing. Note the value must be unique.
- "AZURE_RM_SVC_CONNECTION": Set to the name of the service connection created above.
- "CONTAINER_REGISTRY_NAME": Set to a value of your choosing. Note the value must be unique.
- "KEYVAULT_NAME": Set to a value of your choosing. Note the value must be unique.
- "LOCATION": Set to valid value for the "Name" property for Azure Region hosting the resource group and resources needed to operate the solution.
- "RESOURCE_GROUP_NAME": Set to a value of your choosing. Note the value must be unique.
- "STORAGE_ACCT_NAME": Set to an unique alphanumeric value of your choosing.
- "SUBSCRIPTION_ID": Set to the subscription id for the subscription hosting the Azure Machine Learning workspace.
- "WORKSPACE_NAME": Set to a value of your choosing. Note the value must be unique.

**Terraform only variables.**

- "TFSTATE_RESOURCE_GROUP_NAME": Set to an unique value of your choosing.
- "TFSTATE_STORAGE_ACCT_NAME": Set to an unique alphanumeric value of your choosing.

**Model Deployment Variables.**

**Note Models may be deployed to either batch, online, or both endpoints by setting the properties below. When both are configured to True, the ci pipeline will execute to both endpoints simultaneously.**

- "IS_BATCH_DEPLOYMENT" - Set to True to deploy models to a batch endpoint.
- "IS_ONLINE_DEPLOYMENT" - Set to True to deploy models to an online Endpoint.

**Step 4.** Create Azure Pipelines to deploy the infrastructure, and operate model builds and continuous integration.
Details about how to create a basic Azure Pipeline can be found in [Create your first pipeline](https://learn.microsoft.com/en-us/azure/devops/pipelines/create-first-pipeline?view=azure-devops&tabs).

**Step 5.** Create an azure pipeline to deploy the infrastructure using terraform (_.azure-pipelines/infra/terraform/infra_provision_terraform_pipeline.yml_) yaml files.

**Step 6.** Create one or more Azure Pipelines to setup build validation for either or both of the use cases listed below:

- nyc_taxi
- london_taxi

**Step 7.** Create one or more Azure Pipelines to setup continuous integration for either or both of the use cases listed below:

- nyc_taxi
- london_taxi

**Step 8.** At the Organization Level within Azure Devops, add the Azure DevLabs Machine Learning extension by searching for Machine Learning, clicking the extension pictured below, and then Install on the ensuing page. If you do not have the required permissions, please ask the administrator of the organization to install the extension on your behalf.

![Azure Machine Learning Extension](../media/machinelearningextension.png)

## GitHub Workflows Setup

[Terraform](../../.github/workflows/infra_provision_terraform.yml) Github workflows are provided for deploying the infrastructure.
If the repo is hosted in github its recommended to remove the `azure pipelines @ .azure-pipelines/`

### Creating Service Principal

This command can be used to create a service principal at subscription level:

```bash
# Bash script
az ad sp create-for-rbac --name {NAME_HERE} --role reader --scopes /subscriptions/{SUBSCRIPTION_ID}
```

The output will be in the format of the following:

```json
{
  "appId": "myAppId",
  "displayName": "myServicePrincipalName",
  "password": "myServicePrincipalPassword",
  "tenant": "myTentantId"
}
```

**Note: this will be the only time the password can be seen, so save it.**

### Variable and Secret Configuration

Variables and Secrets need to be configured for the pipelines to run. Variables/secrets can be configured @ 'settings' -> 'secrets and variables' -> 'actions'.

Below is a table detailing the necessary variables/secrets that need to be set.

| Name                    | Secret | Description                                                                            |
| ----------------------- | ------ | -------------------------------------------------------------------------------------- |
| APPINSIGHTS_NAME        | N      | A string compliant with the naming convention for an azure application insights.       |
| CONTAINER_REGISTRY_NAME | N      | A string compliant with the naming convention for an azure container registry.         |
| KEYVAULT_NAME           | N      | A string compliant with the naming convention for an azure key vault.                  |
| LOCATION                | N      | Azure Region                                                                           |
| RESOURCE_GROUP_NAME     | N      | A string compliant with the naming convention for an azure resource group.             |
| STORAGE_ACCT_NAME       | N      | A string compliant with the naming convention for an azure storage account.            |
| WORKSPACE_NAME          | N      | A string compliant with the naming convention for an azure machine learning workspace. |
| ARM_CLIENT_ID           | Y      | The application id corresponding to the service principal created above.               |
| ARM_CLIENT_SECRET       | Y      | The password corresponding to the service principal created above.                     |
| ARM_TENANT_ID           | Y      | The tenant id corresponding to the service principal created above.                    |
| ARM_SUBSCRIPTION_ID     | Y      | The subscription id corresponding to the service principal created above.              |

**Note: these are required for Terraform only.**

| Name                        | Secret | Description                                                                       |
| --------------------------- | ------ | --------------------------------------------------------------------------------- |
| TFSTATE_RESOURCE_GROUP_NAME | N      | Name of the resource group that will contain the terraform state storage account. |
| TFSTATE_STORAGE_ACCT_NAME   | N      | Name of the storage account that will contain the terraform state.                |

Once you have completed this setup, it is a good idea to test out your setup by running an end-to-end test that includes all the steps detailed in [Testing the Initial Setup](./TestInitialSetup.md)
