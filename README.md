# EnvironmentDSC
Create Environment Variables via DSC, from Keyvault secrets.

PowerShell Set Environement Variables __Class based Resource__

__Requirements__
* PowerShell Version 5.0 +
* Server 2012 +

Sample with all values passed in, however I would recommend that you set some as defaults, such as:
- Prefix
- KVName

Prefix is optional.
Prefix should not be part of the keyvault Name, it's only added to the Environment Variable name:
- Defines the category or grouping of the Environment variable.

```powershell
    # sample configuation data

            # KV Secrests Get with Managed Identity - Oauth2
            EnvironmentVarSet           = @(
                @{ Name = 'BotName';        Prefix = 'AzureSettings:'; KVName = 'kvglobal'},
                @{ Name = 'AadAppId';       Prefix = 'AzureSettings:'; KVName = 'kvglobal'},
                @{ Name = 'AadAppSecret';   Prefix = 'AzureSettings:'; KVName = 'kvglobal'},
                @{ Name = 'ServiceDnsName'; Prefix = 'AzureSettings:'; KVName = 'kvglobal'}
            )
```


```powershell
Configuration AppServers
{
    Param (
        [String]$Deployment,
        [String]$clientIDLocal
    )

    node $AllNodes.NodeName
    {
        #-------------------------------------------------------------------
        foreach ($EnvVar in $Node.EnvironmentVarSet)
        {
            EnvironmentDSC $EnvVar.Name
            {
                Name                    = $EnvVar.Name
                Prefix                  = $EnvVar.Prefix
                KeyVaultName            = $EnvVar.KVName
                ManagedIdentityClientID = $clientIDGlobal
            }
            $dependsonEnvironmentDSC += @("[EnvironmentDSC]$($EnvVar.Name)")
        }
```

Full sample available here

- DSC Configuration
    - [ADF/ext-DSC/DSC-AppServers.ps1](https://github.com/brwilkinson/AzureDeploymentFramework/blob/main/ADF/ext-DSC/DSC-AppServers.ps1#L448)
- DSC ConfigurationData
    - [ADF/ext-CD/JMP-ConfigurationData.psd1](https://github.com/brwilkinson/AzureDeploymentFramework/blob/main/ADF/ext-CD/API-ConfigurationData.psd1#L182)

## Invoke the resource directly to sync the files

As well as using DSC in Pull Mode, you can also invoke the resource as part of your Release Pipeline directly
This would be useful to push out a Secret Rotation, where the DSC configuration settings do not change,
however the secret value in the keyvault was updated.

```powershell
$Properties = @{ Name = 'BotName'; KVName = 'acu1-brw-bot-d1-kvglobal'; ManagedIdentityClientID = '47931453-e79d-4d91-bd73-d863f838e28a'}

Invoke-DscResource -Name EnvironmentDSC -Method GET -ModuleName EnvironmentDSC -Property $Properties

Scope                   : Machine
Ensure                  : Present
Name                    : BotName
ManagedIdentityClientID : 47931453-e79d-4d91-bd73-d863f838e28a
KeyVaultName            : acu1-brw-bot-d1-kvglobal
KeyVaultURI             : https://acu1-brw-bot-d1-kvglobal.vault.azure.net

Invoke-DscResource -Name EnvironmentDSC -Method Test -ModuleName EnvironmentDSC -Property $Properties

InDesiredState
--------------
         False

Invoke-DscResource -Name EnvironmentDSC -Method Set -ModuleName EnvironmentDSC -Property $Properties

RebootRequired
--------------
         False
```