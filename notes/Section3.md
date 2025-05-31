# PowerShell and CLI
* No programming (Coding); however there is scripting (Azure CLI, Powershell)

### Where to access the cli
* Azure Portal
* Powershell az
* Bash/CLI
* (bonus) JSON for templates and CLI responses

### Why its needed
* Automation
* Source Control for Sharing scripts
* Reduce errors on repetitive tasks
* can be seen as a form of documentation by looking at how the script executes the desired task.

### How is it tested?
* 1. Performance-based testing ("Labs")
* 2. Code in question

## Study advice
* Powershell/CLI
* Hands on practice while studying - Portal, PS, and CLI
* Leave a lot of time for the labs!!!

## Memorizing Powershell and CLI Commands
- There is predictable naming system for commands

### CLI
- az vm list
- az vm create
- az vm delete

- az keyvault list
- az keyvault create
- az keyvault delete

- az network vnet list
- az network vnet create
- az network vnet delete

- az network vnet subnet list
- az network vnet subnet create
- az network vnet subnet delete

### Powershell
- Get-AzVM
- New-AzVM
- Remove-AzVM

- Get-AzKeyVault
- New-AzKeyVault
- Remove-AzVM

- Get-AzVirtualNetwork
- New-AzVirtualNetwork
- Remove-AzVirtualNetwork

- Get-AzVirtualNetworkSubnetConfig
- New-AzVirtualNetworkSubnetConfig
- Remove-AzVirtualNetworkSubnetConfig

1. Download/update PowerShell
2. Download/update Az Module - Install-Module -Name Az -AllowClobber -Repository PSGallery -Force

Log-In to Azure Powershell: Connect-AzAccount

To list all installed versions of Az: Get-InstalledModule -Name Az -AllVersions | Select-Object -Property Name, Version

To connect to Azure account from Powershell: Connect-AzAccount

To select a specific subscription: Get-AzSubscription -> get Subscription Id -> Set-AzContext -Subscription "Subscription Id"


#
