
# Fetch and Store current Azure Cloud session details, can be referred to later in code.
```
data "azurerm_client_config" "current" {} 
```
# Hub Resource Groups
```
resource "azurerm_resource_group" "hub" {
  name     = var.hub_name
  location = var.location
}
```

# Spoke resource group
```
resource "azurerm_resource_group" "spoke" {
  name     = var.spoke_name
  location = var.location
}
```
# Create log analytics workspace
```
resource "azurerm_log_analytics_workspace" "la" {
  name                = var.hub_name
  location            = var.location
  resource_group_name = azurerm_resource_group.hub.name
  sku                 = "PerGB2018"
}
```

# Provision the Virtual Network and create Peering with the Hub calling vnet module 
```
module "vnet" {
  source = "./modules/vnet"

  hub_name    = var.hub_name
  hub_rg_name = azurerm_resource_group.hub.name

  spoke_name    = var.spoke_name
  spoke_rg_name = azurerm_resource_group.spoke.name

  diag_name = "${var.hub_name}${random_string.random.result}" # Random string generation

  location = var.location
  la_id    = azurerm_log_analytics_workspace.la.id
}
```
# Provision a Keyvault to store Username and Password for Bastion
```
module "kv" {
  source = "./modules/keyvault"

  kv_name             = "${var.hub_name}${random_string.random.result}" # Keyvault name generation by appending random string
  location            = var.location
  resource_group_name = azurerm_resource_group.hub.name
  vm_admin_username   = random_string.user.result
  vm_admin_password   = random_password.pw.result
}
```

# Creating all Bastion VMs calling all modules
```
module "vm" {
  source = "./modules/vm"

  resource_group_name = azurerm_resource_group.hub.name
  location            = var.location
  bastion_subnet_id   = module.vnet.bastion_subnet_id
  kv_id               = module.kv.kv_id
  vm_subnet_id        = module.vnet.vm_subnet_id
}
```

# Creating the supporting infra like ACR,Cosmosdb,etc. (This block is optional and can be skipped. ) 
```
module "supporting" {
  source = "./modules/supporting"

  location                   = var.location
  hub_vnet_id                = module.vnet.hub_vnet_id
  spoke_vnet_id              = module.vnet.spoke_vnet_id
  private_endpoint_subnet_id = module.vnet.private_endpoint_subnet_id
  spoke_rg_name = azurerm_resource_group.spoke.name
  hub_rg_name = azurerm_resource_group.hub.name

  depends_on = [
    module.vnet
  ]
}
```

# Creating a service principal for Openshift
```
module "serviceprincipal" {
  source = "./modules/serviceprincipal"

  aro_spn_name = var.aro_spn_name
  spoke_rg_name = azurerm_resource_group.spoke.name
  hub_rg_name = azurerm_resource_group.hub.name

  depends_on = [
    module.vnet
  ]
}
```

# Provisions the openshift cluster using modules
```
module "aro" {
  source = "./modules/aro"

  location = var.location

  spoke_vnet_id = module.vnet.spoke_vnet_id
  master_subnet_id = module.vnet.master_subnet_id
  worker_subnet_id = module.vnet.worker_subnet_id

  sp_client_id = module.serviceprincipal.sp_client_id
  sp_client_secret = module.serviceprincipal.sp_client_secret
  aro_rp_object_id = var.aro_rp_object_id
  spoke_rg_name = azurerm_resource_group.spoke.name
  base_name = var.aro_base_name
  domain = var.aro_domain

  depends_on = [
    module.serviceprincipal
  ]
}
```

# Create a Frontdoor LB for Openshift
```
module "frontdoor" {
  source = "./modules/frontdoor"

  location = var.location
  aro_worker_subnet_id = module.vnet.worker_subnet_id
  la_id = azurerm_log_analytics_workspace.la.id
  random = random_string.random.result
  aro_resource_group_name = module.aro.cluster_resource_group_name
  spoke_rg_name = azurerm_resource_group.spoke.name
  
  depends_on = [
    module.aro
  ]
}
```

# Provsion Container Insights for debugging container.
```
module "containerinsights" {
  source = "./modules/containerinsights"

  location = azurerm_log_analytics_workspace.la.location
  workspace_resource_id = azurerm_log_analytics_workspace.la.id
  workspace_name = azurerm_log_analytics_workspace.la.name
  spoke_rg_name = azurerm_resource_group.hub.name
}
```
