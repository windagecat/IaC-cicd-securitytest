# Release pipeline
With Releases for Azure Pipeline, create the release pipeline.

![8](./images/8.png)
## Continuous deployment trigger
Set continuous deployment trigger in Artifacts.

![release_1](./images/release_1.png)


## Stages
Set up Stages as shown below.

- Stage 

![release_2](./images/release_2.png)

- Agent job

![release_3](./images/release_3.png)


- Pipeline variables
> If you've editted file terraform.vars for remote-tfstate of product environment, you need to change value of variable RG.

![release_4](./images/release_4.png)


- Pipeline yaml for tasks
> Workload identity federation (OIDC) is recommended for service connection
```bash
variables:
  RG: '<resource group name>'
  cluster_name: 'aks_product_cp01'
  acr_name: 'acrproduct'

steps:
- task: ms-devlabs.custom-terraform-tasks.custom-terraform-installer-task.TerraformInstaller@1
  displayName: 'Install Terraform 1.6.6'
  inputs:
    terraformVersion: 1.6.6
  timeoutInMinutes: 5

- task: ms-devlabs.custom-terraform-tasks.custom-terraform-release-task.TerraformTaskV4@4
  displayName: 'Terraform init'
  inputs:
    workingDirectory: '<working directory>'
    backendServiceArm: '<service connection>' #Workload identity federation (OIDC) is recommended 
    backendAzureRmResourceGroupName: '<resource group name>'
    backendAzureRmStorageAccountName: <StorageAccountName>
    backendAzureRmContainerName: 'azure-tfstate'
    backendAzureRmKey: terraform.tfstate
  timeoutInMinutes: 10

- task: ms-devlabs.custom-terraform-tasks.custom-terraform-release-task.TerraformTaskV4@4
  displayName: 'Terraform validate'
  inputs:
    command: validate
    workingDirectory: '<working directory>'
  timeoutInMinutes: 10

- task: ms-devlabs.custom-terraform-tasks.custom-terraform-release-task.TerraformTaskV4@4
  displayName: 'Terraform plan'
  inputs:
    command: plan
    workingDirectory: '<working directory>'
    commandOptions: '-var="resource_group_name=$(RG)" -var="aks_cluster_name=$(cluster_name)" -var="acr_name=$(acr_name)"'
    environmentServiceNameAzureRM: '<service connection>' #Workload identity federation (OIDC) is recommended 
  timeoutInMinutes: 10

- task: ms-devlabs.custom-terraform-tasks.custom-terraform-release-task.TerraformTaskV4@4
  displayName: 'Terraform apply'
  inputs:
    command: apply
    workingDirectory: '<working directory>'
    commandOptions: '-var="resource_group_name=$(RG)" -var="aks_cluster_name=$(cluster_name)" -var="acr_name=$(acr_name)"'
    environmentServiceNameAzureRM: '<service connection>' #Workload identity federation (OIDC) is recommended 
  timeoutInMinutes: 30

```
- Triggers

![5](./images/5.png)

- Gates(Post-deployment conditions)
> This is Compliance Assessment setting.<br>
It is necessary to assign policies or initiatives (such as ASC Default) to the target scope beforehand.


![6](./images/6.png)

![7](./images/7.png)


