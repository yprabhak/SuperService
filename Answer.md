#Super Service CI

This repo contains the template to build, test and create container for dotnet core application.
azure-pipeline.yml file is a yaml file used for creating the ci pipeline in azure devops
 steps included in azure-pipeline.yml
   1. Install dotnet sdk
   2. Run dotnet restore using dotnet CLI task
   3. Run dotnet build using dotnet CLI task
   4. Run dotnet test using dotnet CLI task
   5. Create docker container using docker commands in a poweshell task.

Above dotnet tasks are internally running the dotnet cli commands. 
You can use cmd, powershell, or bash also for running the above dotnet commands -> dotnet restore, build and test.

#Super Service CD
Here I am using Azure Cloud Service Provider,
So we have to create, 
  Azure Container Registry for store container images.
  Azure Kubernetes Service for Container orchestration.
a
You can create ACS and AKS either manually or using scripts (Azure CLI).

you can install azure cli from microsoft - https://learn.microsoft.com/en-us/cli/azure/install-azure-cli

After installing Azure CLI you can run az commands local,

same way in azure pipelines you can use azure cli tasks and run az commands, -> this way you can automate the acre creation.
Also you can use terraform templates for provision ACR and AKS.

if you are running the scripts in local, First you have to authenticate to azure environment by running, "az login" command.

In Azure DevOps you can create an Azure Resource Manager service connection to azure using service principle. and in Azure CLI task you can use this service connection to directly authenticate to the resource group under your subscription.

Scripts,

 1. az group create --name AzureRG --location eastus
 2. az acr create --resource-group AzureRG --name dotnetaksdemo --sku Standard - SKU standard is required for VNet integration or private endpoint integration.

You can use terraform also to create the azure resources.
  https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/container_registry
In azure pipelines after defining the tf file, you can use, terraform tasks to init, plan, apply or destroy the infra.

After creating the ACR you can use the docker commands or docker tasks to build and push the container image. 
If you are using docker task make sure to create the docker registry service connection in Azure DevOps and use that in the docker tasks.
If you are using Azure CLI powershell you have to login to the docker registry and then you have to tag the image with acr-name/repo:tag format use docker tag and you can push the image using docker push command.


