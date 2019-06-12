# Create a Static IP Address for Azure Kubernetes Environment
> Quick instructions on how to setup a static IP address that can be used by your Kubernetes environment

## Table of Contents

- [Configuration Details](#configuration-details)
- [Minimum Requirements](#minimum-requirements)
- [Create Static IP Address](#create-static-ip-address)
- [Related Cheatsheets](#related-cheatsheets)
- [Related Links](#related-links)

## Configuration Details

The following are config values you will need to create the static IP address.

> NOTE: Reference the following [cheatsheet](azure-kubernetes-setup.md) if you are not sure what these values are

- ***Resource Group Name***: Used to group together resources that make up your Kubernetes environment
- ***Cluster Name***: The name of your Kubernetes cluster
- ***IP Name***: A name to identify the static IP address that will be created
    - A best practice is to use `camelCase` and prefix the name with `ip`
    - e.g. `ipIngress`

## Minimum Requirements

* [Azure Kuberetes Service Instance](https://azure.microsoft.com/en-us/services/kubernetes-service)
* [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
* [Kubernetes CLI - kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

> ***NOTE:*** Unless told otherwise, all instructions happen via a Terminal or Command Prompt

> ***NOTE:*** All values below in curly brackets `{}` are placeholders and need to be replaced with [configuration values](#configuration-details) mentioned above. e.g. `{resourceGroupName}` needs to be replaced with `rgAcmeDev`.

## Create Static IP Address

The following will first retrieve a programmatic version of the Resource Group name of your Kuberenetes Cluster. You will then use the retrieved name to create the IP Address.

```bash
az aks show --resource-group "{resourceGroupName}" --name "{clusterName}" --query nodeResourceGroup -o tsv # This returns a {programmaticResourceGroupName} that will be used below

az network public-ip create --resource-group "{programmaticResourceGroupName}" --name "{ipName}" --allocation-method static
```

The last command will return the IP address and additional info related to the IP.

## Related Cheatsheets
* [Microsoft Azure Kubernetes Setup](azure-kubernetes-setup.md)

## Related Links
* [Azure Kubernetes Service](https://azure.microsoft.com/en-us/services/kubernetes-service)