# Accessing Azure Key Vault Secrets using Azure DevOps Pipelines 

## 1) Create Azure Key Vault with Secrets and persmissions through portal or using below Az-cli commands/Scripts 

The following script will create a Key Vault and Secret:

```bash
# create the variables
KEYVAULT_RG="rg-keyvault-devops"
KEYVAULT_NAME="keyvault019"
SUBSCRIPTION_ID=$(az account show --query id -o tsv)

# create new resource group from the portal or using below Az-cli command
az group create -n rg-keyvault-devops -l westeurope

# create key vault with RBAC option (not Access Policy) 
az keyvault create --name $KEYVAULT_NAME \
   --resource-group $KEYVAULT_RG \
   --enable-rbac-authorization
```

```bash

# On portal 
# 1. Assign RBAC role to the current user to manage secrets 
# 2. create a secret
# 3. Create Service Principal to access Key Vault from Azure DevOps Pipelines


## 3) Create a pipeline to access Key Vault Secrets

### 3.1) Create Service Connection using the SPN

Create a service connection in Azure DevOps using the SPN created earlier.

### 3.2) Create YAML pipeline

Create the following yaml pipeline to get access to the secrets.

```yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- task: AzureKeyVault@2
  displayName: Get Secrets from Key Vault
  inputs:
    azureSubscription: 'spn-keyvault-devops'
    KeyVaultName: 'keyvault019'
    SecretsFilter: '*' # 'DatabasePassword'
    RunAsPreJob: false

- task: CmdLine@2
  displayName: Write Secret into File
  inputs:
    script: |
      echo $(DatabasePassword)
      echo $(DatabasePassword) > secret.txt
      cat secret.txt

- task: CopyFiles@2
  displayName: Copy Secrets File
  inputs:
    Contents: secret.txt
    targetFolder: '$(Build.ArtifactStagingDirectory)'

- task: PublishBuildArtifacts@1
  displayName: Publish Secrets File
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
    publishLocation: 'Container'
```

  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
    publishLocation: 'Container'
```
