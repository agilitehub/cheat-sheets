# Microsoft Azure Kubernetes Setup
> Step by step instructions on how to get up and running with Kubernetes on Microsoft Azure Cloud

## Table of Contents

- [Configuration Details](#configuration-details)
- [Minimum Requirements](#minimum-requirements)
- [Once Off Setups](#once-off-setups)
    - [Login to Azure](#login-to-azure)
    - [Create Resource Group](#create-resource-group)
    - [Create Container Registry](#create-container-registry)
    - [Create Service Principal](#create-service-principal)
    - [Create Kubernetes Cluster Without AD Enabling AKS](#create-kubernetes-cluster-without-ad-enabling-aks)
    - [Add Credentials to KubeConfig](#add-credentials-to-kubeconfig)
    - [Create Cluster Role Binding and Launch Dashboard](#create-cluster-role-binding-and-launch-dashboard)
- [Delete Kubernetes Cluster and All Group Resources](#delete-kubernetes-cluster-and-all-group-resources)
- [Related Cheatsheets](#related-cheatsheets)
- [Related Links](#related-links)

## Configuration Details

The following are config values you will need to use to create the necessary resources

- ***Resource Group Name***: Used to group together resources that make up your Kubernetes implementation
    - A best practice is to use `camelCase` and prefix the name with `rg` (i.e. rg=Resource Group)
    - e.g. `rgAcmeDev`
- ***Location***: The [location](https://azure.microsoft.com/en-us/global-infrastructure/locations/) of the Azure data center that will host your Kubernetes cluster
- ***Azure Container Registry Name***: This is where container images will be hosted
    - The name must be in `lowercase`
    - A best practice prefix the name with `acr` (i.e. acr=Azure Container Registry)
    - e.g. `acracmedev`
- ***ACR SKU***: The type of SKU to be used for the Azure Container Registry
    - Basic (Cost effective)
    - Standard (Satisfies most production needs)
    - Premium
- ***Node Count***: A Number value representing the amount of nodes to be created
- ***Cluster Name***: The name of the Kubernetes cluster you are creating
    - A best practice is to use `camelCase` and prefix the name with `cluster`
    - e.g. `clusterAcmeDev`

## Minimum Requirements

* [Microsoft Azure Account](https://azure.microsoft.com/en-us/free/?WT.mc_id=A261C142F)
* [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
* [Kubernetes CLI - kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

> ***NOTE:*** Unless told otherwise, all instructions happen via a Terminal or Command Prompt

> ***NOTE:*** All values below in curly brackets `{}` are placeholders and need to be replaced with [configuration values](#configuration-details) mentioned above. e.g. `{resourceGroupName}` needs to be replaced with `rgAcmeDev`.

## Once Off Setups

This section focuses on once-off setups to get the Kubernetes environment up and running

### Login to Azure

```bash
az login
```
This takes you to a browser page to complete the authentication. You can close the page once authentication is successful.

### Create Resource Group

```bash
az group create --name "{resourceGroupName}" --location "{locationName}"
```

### Create Container Registry

```bash
az acr create --resource-group "{resourceGroupName}" --name "{acrname}" --sku "{skuType}" --location "{locationName}"
```

### Create Service Principal

```bash
az ad sp create-for-rbac --skip-assignment #NB: Take note of App Id and Password as they get used in the next few commands

ACR_ID=$(az acr show --name "{acrname}" --resource-group "{resourceGroupName}" --query "id" --output tsv) #In Powershell, replace ACR_ID with $env:ACR_ID

az role assignment create --assignee "{appId}" --scope $ACR_ID --role acrpull #In Powershell, replace $ACR_ID with $env:ACR_ID
```

### Create Kubernetes Cluster Without AD Enabling AKS

```bash
az aks create --resource-group "{resourceGroupName}" --node-count "{nodeCount}" --name "{clusterName}" --location "{locationName}" --service-principal "{appId}" --client-secret "{password}" --node-vm-size "Standard_DS1_v2" --generate-ssh-keys --enable-addons "monitoring" #Takes about 10 minutes to run
```

### Add Credentials to KubeConfig

```bash
az aks get-credentials --resource-group "{resourceGroupName}" --name "{clusterName}" --overwrite-existing
```

### Create Cluster Role Binding and Launch Dashboard

1. Create cluster role bindings for Kubernetes Dashboard and Default Namespace:

```bash
kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard

kubectl create clusterrolebinding default --clusterrole=cluster-admin --serviceaccount=kube-system:default
```

2. Launch Kubernetes Dashboard

```bash
az aks browse --resource-group "{resourceGroupName}" --name "{clusterName}"
```

## Delete Kubernetes Cluster and All Group Resources

> ***NB***: Please be careful of this, as this cannot be undone

```bash
az group delete --name "{resourceGroupName}" --yes --no-wait

az ad sp delete --id $(az aks show -g "{resourceGroupName}" -n "{clusterName}" --query servicePrincipalProfile.clientId -o tsv)

```

On your local environment, remove cluster credentials from Kube Config. The easiest way to remove them is via VS Code's [Kubernetes](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools) extension.

## Related Cheatsheets
* [Create a Static IP Address for Azure Kubernetes Environment](azure-kubernetes-static-ip.md)

## Related Links
* [Microsoft Azure](https://azure.microsoft.com/en-us)
* [Kubernetes](https://kubernetes.io)
* [Azure Kubernetes Service](https://azure.microsoft.com/en-us/services/kubernetes-service)
* [Azure Kubernetes Service Documentation](https://docs.microsoft.com/en-us/azure/aks)
* [Azure Container Registry SKUs](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-skus)
* [Azure Speed](http://www.azurespeed.com)
* [Azure Pricing](https://azureprice.net)
* [VS Code](https://code.visualstudio.com)
* [VS Code Kubernetes Extension](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools)