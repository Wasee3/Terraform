
# Variables

### Contents of file are explained vars.tf are explained here

```
variable "tenant_id" 
{
  type = string
} # Required variables


variable "subscription_id" {
  type = string
} # Required variables

variable "location" {
  type    = string
  default = "eastus"
} # Optional variables

variable "hub_name" {
  type    = string
  default = "hub-aro"
} # Optional variables

variable "spoke_name" {
  type    = string
  default = "spoke-aro"
} # Optional variables

variable "aro_spn_name" {
  type    = string
  default = "aro-lza-sp"
} # Optional variables

resource "random_password" "pw" {
  length      = 16
  special     = true
  min_lower   = 3
  min_special = 2
  min_upper   = 3

  keepers = {
    location = var.location
  }
} # Generating a random password

resource "random_string" "user" {
  length  = 16
  special = false

  keepers = {
    location = var.location
  }
} # Generating random string

resource "random_string" "random" {
  length = 6
  special = false
  min_lower = 3
  min_upper = 1

  keepers = {
    location = var.location
  }
} # Generating random string

variable "aro_rp_object_id" {
  type = string
} # Required variables

variable "aro_base_name" {
  type = string
} # Required variables

variable "aro_domain" {
  type = string
} # Required variables

```
