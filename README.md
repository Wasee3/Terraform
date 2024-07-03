# Terraform Code

Purpose of this document is to explain the terraform code for Provisioning Azure Openshift Landing Zone.

ARO deployment is done through ARM template.

The rest of the landing zones used is provisioned through Terraform.

  Folder Structure
```
  terraform
  |--documentation (This directory conatains all the necessary documentation to understand the code.)
  |--modules (This directory contains all the modules.)
     |--aro (Terraform code to provision Openshift Cluster)
     |--containerinsights (Terraform code to provision and configure Container Insights.)
     |--frontdoor (Terraform code to provision frontdoor LB.)
     |--keyvault (Terraform code to provision Keyvault.)
     |--serviceprincipal (Terraform code to provision service principals.)
     |--supporting (Terraform code to provision supporting resources like ACR,CosmosDB,etc.)
     |--vm (Terraform code to provision Bastion vms in the Hub vnet.)
     |--vnet (Terraform code provision a vnet and create a vnet Peering.)
  |--post_deployment (This document contains commands for post deployment config.)
  |--main.tf (Main file that contains all the code.)
  |--provider.tf (Provider details required by main.tf)
  |--variables.tf (All global main.tf variable declarations. Some Variables referred in modules are defined separately within the module directory. )
```

Scrolling through huge Terraform code to find the right resource to fix makes life hard.

With modules code can be broken down into smaller snippets, which increases readability and maintainance of the code.

