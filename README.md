# Azure Managed Mysql Service 
[![Changelog](https://img.shields.io/badge/changelog-release-green.svg)](CHANGELOG.md) [![Notice](https://img.shields.io/badge/notice-copyright-yellow.svg)](NOTICE) [![Apache V2 License](https://img.shields.io/badge/license-Apache%20V2-orange.svg)](LICENSE) [![TF Registry](https://img.shields.io/badge/terraform-registry-blue.svg)](https://registry.terraform.io/modules/claranet/db-mysql/azurerm/)

This Terraform module creates an [Azure MySQL server](https://www.terraform.io/docs/providers/azurerm/r/mysql_server.html) 
with [databases](https://www.terraform.io/docs/providers/azurerm/r/mysql_database.html)  and associated admin users along with logging activated and 
[firewall rules](https://www.terraform.io/docs/providers/azurerm/r/mysql_firewall_rule.html).

## Requirements

* [AzureRM Terraform provider](https://www.terraform.io/docs/providers/azurerm/) >= 1.31
* [MySQL Terraform provider](https://www.terraform.io/docs/providers/mysql/) >= 1.6

## Terraform version compatibility

| Module version | Terraform version |
|----------------|-------------------|
| >= 2.x.x       | 0.12.x            |
| < 2.x.x        | 0.11.x            |

## Usage

This module is optimized to work with the [Claranet terraform-wrapper](https://github.com/claranet/terraform-wrapper) tool
which set some terraform variables in the environment needed by this module.
More details about variables set by the `terraform-wrapper` available in the [documentation](https://github.com/claranet/terraform-wrapper#environment).

```hcl
module "azure-region" {
  source  = "claranet/regions/azurerm"
  version = "x.x.x"

  azure_region = var.azure_region
}

module "rg" {
  source  = "claranet/rg/azurerm"
  version = "x.x.x"

  location    = module.azure-region.location
  client_name = var.client_name
  environment = var.environment
  stack       = var.stack
}

module "mysql" {
  source  = "claranet/db-mysql/azurerm"
  version = "x.x.x"

  client_name          = var.client_name
  environment          = var.environment
  location             = module.azure-region.location
  location_short       = module.azure-region.location_short
  resource_group_name  = module.rg.resource_group_name
  stack                = var.stack

  tier     = "GeneralPurpose"
  capacity = 4

  allowed_cidrs = ["10.0.0.0/24", "12.34.56.78/32"]

  server_storage_profile = {
    storage_mb            = 5120
    backup_retention_days = 10
    geo_redundant_backup  = "Enabled"
  }

  administrator_login    = var.administrator_login
  administrator_password = var.administrator_password
  databases_names        = ["my_database"]

  force_ssl             = true
  mysql_options         = [{name="interactive_timeout", value="600"}, {name="wait_timeout", value="260"}]
  mysql_version         = "5.7"
  databases_charset     = {
    "my_database" = "utf8"
  }
  databases_collation   = {
    "my_database" = "utf8_general_ci"
  }

  extra_tags = var.extra_tags
}
```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:-----:|
| administrator\_login | MySQL administrator login | `string` | n/a | yes |
| administrator\_password | MySQL administrator password. Strong Password : https://docs.microsoft.com/en-us/sql/relational-databases/security/strong-passwords?view=sql-server-2017 | `string` | n/a | yes |
| allowed\_cidrs | List of authorized cidrs | `list(string)` | n/a | yes |
| allowed\_subnets | List of authorized subnet ids | `list(string)` | `[]` | no |
| capacity | Capacity for MySQL server sku : https://www.terraform.io/docs/providers/azurerm/r/mysql_server.html#capacity | `number` | `4` | no |
| client\_name | Name of client | `string` | n/a | yes |
| create\_databases\_users | True to create a user named <db>(_user) per database with generated password. | `bool` | `true` | no |
| custom\_server\_name | Custom Server Name identifier | `string` | `""` | no |
| databases\_charset | Valid mysql charset : https://dev.mysql.com/doc/refman/5.7/en/charset-charsets.html | `map(string)` | `{}` | no |
| databases\_collation | Valid mysql collation : https://dev.mysql.com/doc/refman/5.7/en/charset-charsets.html | `map(string)` | `{}` | no |
| databases\_names | List of databases names | `list(string)` | n/a | yes |
| enable\_logs\_to\_log\_analytics | Boolean flag to specify whether the logs should be sent to Log Analytics | `bool` | `false` | no |
| enable\_logs\_to\_storage | Boolean flag to specify whether the logs should be sent to the Storage Account | `bool` | `false` | no |
| environment | Name of application's environnement | `string` | n/a | yes |
| extra\_tags | Map of custom tags | `map(string)` | `{}` | no |
| force\_ssl | Force usage of SSL | `bool` | `true` | no |
| location | Azure location for Key Vault. | `string` | n/a | yes |
| location\_short | Short string for Azure location. | `string` | n/a | yes |
| logs\_log\_analytics\_workspace\_id | Log Analytics Workspace id for logs | `string` | `""` | no |
| logs\_storage\_account\_id | Storage Account id for logs | `string` | `""` | no |
| logs\_storage\_retention | Retention in days for logs on Storage Account | `string` | `"30"` | no |
| mysql\_options | List of configuration options : https://docs.microsoft.com/fr-fr/azure/mysql/howto-server-parameters#list-of-configurable-server-parameters | `list(map(string))` | `[]` | no |
| mysql\_version | Valid values are 5.6 and 5.7 | `string` | `"5.7"` | no |
| name\_prefix | Optional prefix for PostgreSQL server name | `string` | `""` | no |
| resource\_group\_name | Name of the application ressource group, herited from infra module | `string` | n/a | yes |
| server\_storage\_profile | Storage configuration : https://www.terraform.io/docs/providers/azurerm/r/mysql_server.html#storage_profile | `map(string)` | <pre>{<br>  "backup_retention_days": 10,<br>  "geo_redundant_backup": "Enabled",<br>  "storage_mb": 5120<br>}<br></pre> | no |
| stack | Name of application stack | `string` | n/a | yes |
| tier | Tier for MySQL server sku : https://www.terraform.io/docs/providers/azurerm/r/mysql_server.html#tier Possible values are: GeneralPurpose, Basic, MemoryOptimized | `string` | `"GeneralPurpose"` | no |
| enable\_user\_suffix | True to append a _user suffix to database users | `bool` | `true` | no |

## Outputs

| Name | Description |
|------|-------------|
| mysql\_administrator\_login | Administrator login for MySQL server |
| mysql\_configuration\_id | The list of all configurations resource ids |
| mysql\_database\_ids | The list of all database resource ids |
| mysql\_databases\_names | List of databases names |
| mysql\_databases\_users | List of usernames of created users corresponding to input databases names. |
| mysql\_databases\_users\_passwords | List of passwords of created users corresponding to input databases names. |
| mysql\_firewall\_rule\_ids | List of MySQL created rules |
| mysql\_fqdn | FQDN of the MySQL server |
| mysql\_server\_id | MySQL server ID |
| mysql\_server\_name | MySQL server name |
| mysql\_vnet\_rule\_ids | The list of all vnet rule resource ids |

## Related documentation

Terraform Azure MySQL Server documentation: [www.terraform.io/docs/providers/azurerm/r/mysql_server.html](https://www.terraform.io/docs/providers/azurerm/r/mysql_server.html)

Terraform Azure MySQL Database documentation: [www.terraform.io/docs/providers/azurerm/r/mysql_database.html](https://www.terraform.io/docs/providers/azurerm/r/mysql_database.html)

Microsoft Azure documentation: [docs.microsoft.com/fr-fr/azure/mysql/overview](https://docs.microsoft.com/fr-fr/azure/mysql/overview)
