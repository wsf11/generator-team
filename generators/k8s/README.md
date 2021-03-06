# Kubernetes sub-generator

The k8s sub-generator creates a CI/CD pipeline in Visual Studio Teams Services (VSTS) and generates the template files needed to provision a basic Kubernetes web application using helm.

#### Table of Contents
1. [Installing](#installing)
2. [Prerequisites](#prerequisites)
3. [Getting Started](#gettingStarted)
4. [Usage](#usage)

## Installing <a name="installing"></a>

Please refer to the [home](https://github.com/DarqueWarrior/generator-team) repository to install the Yeoman generator 

## Prerequisites <a name="prerequisites"></a>

Along with an Azure Subscription and a VSTS Project, a few things have to be provisioned before running the generator: 

1. A Kubernetes Cluster in either [Azure Container Services](https://docs.microsoft.com/en-us/azure/container-service/kubernetes/) (ACS) or [Azure Kubernetes Services](https://azure.microsoft.com/en-us/services/kubernetes-service/) (AKS)

2. An [Azure Container Registry](https://azure.microsoft.com/en-us/services/container-registry/) (ACR)


3. A [Service Principal](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-auth-aks?toc=%2fazure%2faks%2ftoc.json) in Azure allowing your Kubernetes Cluster and your ACR to communicate

   - If an Azure account is not configured with the CLI, run the following and follow the prompts:
   
      ```
      az login
      ```

   - The service principal can now be created by running the following:

      ```
      #!/bin/bash

      ACR_NAME=<my-acr-instance>
      SERVICE_PRINCIPAL_NAME=<acr-service-principal>

      # Populate the ACR login server and resource id.
      ACR_LOGIN_SERVER=$(az acr show --name $ACR_NAME --query loginServer --output tsv)
      ACR_REGISTRY_ID=$(az acr show --name $ACR_NAME --query id --output tsv)

      # Create a contributor role assignment with a scope of the ACR resource.
      SP_PASSWD=$(az ad sp create-for-rbac --name $SERVICE_PRINCIPAL_NAME --role Reader --scopes $ACR_REGISTRY_ID --query password --output tsv)

      # Get the service principle client id.
      CLIENT_ID=$(az ad sp show --id http://$SERVICE_PRINCIPAL_NAME --query appId --output tsv)

      # Output used when creating Kubernetes secret.
      echo "Service principal ID: $CLIENT_ID"
      echo "Service principal password: $SP_PASSWD"
      ```

4. The K8s Image Pull Secret
   
   - To configure Kubectl to talk to the cluster:

      | K8s Service | Install | Get Credentials |
      | --- | --- | --- |
      | AKS | `az aks install-cli` | `az aks get-credentials --name=<myClusterName> --resource-group=<myResourceGroup>` |
      | ACS |  | `az acs kubernetes get-credentials --name=<myClusterName> --resource-group=<myResourceGroup>` |

   - To create the pull secret, run:

      ```
      # Create image pull secret
      MY_PULL_SECRET=<acr-auth>

      kubectl create secret docker-registry $MY_PULL_SECRET --docker-server $ACR_LOGIN_SERVER --docker-username $CLIENT_ID --docker-password $SP_PASSWD --docker-email <email-address>
      ```

5. A VSTS [personal access token](https://docs.microsoft.com/en-us/vsts/organizations/accounts/use-personal-access-tokens-to-authenticate?view=vsts) (PAT) 


## Getting Started <a name="gettingStarted"></a>
To start the sub-generator, run

   ```
   yo team:k8s
   ```

## Usage <a name="usage"></a>
The Generator will prompt you to:

1. Enter the name of the **VSTS profile**. 
   
   - Denoted by **{account}** in `{account}.visualstudio.com`

2. Enter the **PAT** denoted with the profile
   
   - The PAT can be found under the security tab in the VSTS profile or by going to `https://{account}.visualstudio.com/_details/security/tokens`

3. Select the preferred **agent queue**

   - This will be the agent that runs the build and release tasks

4. Enter the name of the **VSTS application**

   - Denoted by **{applicationName}** in `https://{account}.visualstudio.com/{applicationName}`

5. Select to deploy to either **AKS** or **ACS**

6. Select an **Azure subscription**

7. Enter the name of the **azure container registry** that the images will be pulled from, and the **resource group** the container is located in

   - Both can be found by running:

      ```
      az acr list 
      ```
8. Enter the name of the **image pull secret**
   
   - The name can be found by running:

      ```
      kubectl get secrets
      ```

9. Enter the **name** and **resource group** of your Kubernetes Cluster

   - Both can be found by running:

      ```
      az aks list
      ```

10. Enter the path of your Kubernetes Config file
    
      - Enter the path to the config file. For example, if the config file is located in the `.kube` directory, enter:
   
         ```
         /home/user/.kube
         ```
