# Super Service CI

This repo contains the template to build, test and create container for dotnet core application.
azure-pipeline.yml file is a yaml file used for creating the ci pipeline in azure devops
 steps included in azure-pipeline.yml
 Note: **pipelines** folder contains the CICD templates,
   1. Install dotnet sdk
   2. Run dotnet restore using dotnet CLI task
   3. Run dotnet build using dotnet CLI task
   4. Run dotnet test using dotnet CLI task
   5. Create docker container using docker commands in a poweshell task.

Above dotnet tasks are internally running the dotnet cli commands. 
You can use cmd, powershell, or bash also for running the above dotnet commands -> dotnet restore, build and test.

# Super Service CD
Here I am using Azure Cloud Service Provider,
So we have to create, 
  Azure Container Registry for store container images.
  Azure Kubernetes Service for Container orchestration.
You can create ACS and AKS either manually or using scripts (Azure CLI).

you can install azure cli from microsoft - https://learn.microsoft.com/en-us/cli/azure/install-azure-cli

After installing Azure CLI you can run az commands local,

same way in azure pipelines you can use azure cli tasks and run az commands, -> this way you can automate the acre creation.
Also you can use terraform templates for provision ACR and AKS.

if you are running the scripts in local, First you have to authenticate to azure environment by running, "az login" command.

In Azure DevOps you can create an Azure Resource Manager service connection to azure using service principle. and in Azure CLI task you can use this service connection to directly authenticate to the resource group under your subscription.

Scripts,

 1. az group create --name uper-service-rg --location eastus --> This will create Resource Group
 2. az acr create --resource-group super-service-rg --name superserviceacr --sku Standard
 Note - SKU standard is required for VNet integration or private endpoint integration.

You can use terraform also to create the azure resources.
  https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/container_registry
In azure pipelines after defining the tf file, you can use, terraform tasks to init, plan, apply or destroy the infra.

After creating the ACR you can use the docker commands or docker tasks to build and push the container image. 
If you are using docker task make sure to create the docker registry service connection in Azure DevOps and use that in the docker tasks.
If you are using Azure CLI or Powershell you have to login to the docker registry and then you have to tag the image with acr-name/repo:tag format, you can use docker tag command to tag image, and you can push the image using docker push command.

## AKS Deployment. 
To create an AKS cluster you can either use azure portal or you can use scripts, you can use Azure CLI commands to create and AKS cluster,
example : 
- az aks create --resource-group super-service-rg --name superserviceaks \
   --node-count 2 --enable-addons http_application_routing \
   --generate-ssh-keys --service-principal <SERVICE_PRINCIPAL_ID> \
   --client-secret <SERVICE_PRINCIPAL_PASSWORD> \
   --attach-acr superserviceacr

This is a basic example, we can use terraform script to provision AKS. 
Important things to remember while provisioning AKS cluster about the type of networking: By default AKS uses kubenet network, If you are creating an AKS cluster for a large scale business application it is better to create your own Vnet and create AKS with Azure CNI networking. Here you can create a vnet with name "internal-assets" and create subnet based on IP requirement. 

After Provisioning AKS cluster before application deployment you have to make sure all the required access you added to azure resources to communicate in Vnet. For example if you are accessing container images you have to connect with ACR. So as above example you have to create a service principle and you have to give access to that Service Prinicple to the AKS cluster and ACR. When we are attaching AKS cluster to VNet you have to assign network related role permissions.

Connect To AKS Cluster for Deployment: 
You can connect to aks cluster from local machine or you can use the kubernetes task in azure devops for deploying the app.
Locally, 
 - az login - login to the correct subscription - if you have multiple subscription set your subscription using az account set command.
Getting credentials to access the cluster,  
 - az aks get-credentials -g super-service-rg -n superserviceaks --admin 
Use admin for getting the user admin role for current user.
To automate this process in Azure DevOps you can create the Kubernetes service connection using the credentials created.

Deployment manifest files description - 
You can see the manifests files created under pipelines folder for AKS deployment.
- manifest.yml - application deployment manifest file, contains the service deployment also (CLusterIP)
- nginx-ingress.yaml - create the ingress-controller - here we are using NGINX ingress controller.
- superserviceingress.yaml - 
- HPA - Horzontal pode auto scaling

  
Alerts and Monitor 
You can setup alerts and montor to proactively monitor the AKS cluster node, pods etc.
Few options,
- Configure Azure Alerts and Monitor.
- You can enable Diagnostics settings in azure for sending telemetry data to specific endpoints like Log Analytics Workspace or Eventhub etc.
- Setup Grafana : Configure a grafana dashboard - you can enable azure managed grafana or you can create your own grafana org and send Telemetry data to Gragana using Open Telemetry collector configuration.

For Alerts - you can create in Grafana Alerts based on cetain conditions we set like CPU percentage, memory utilisation or pod failures etc, Or You can create Alert in azure and create action group to inform group of people about failures and all.

