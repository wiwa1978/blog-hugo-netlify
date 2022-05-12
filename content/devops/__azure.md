### Downloading the container

```
✘ wauterw@WAUTERW-M-65P7  ~  docker pull mcr.microsoft.com/azure-powershell
```

### Running the container

```
wauterw@WAUTERW-M-65P7  ~  docker run -it mcr.microsoft.com/azure-powershell pwsh
wauterw@WAUTERW-M-65P7  ~  docker run -it mcr.microsoft.com/azure-powershell pwsh
PowerShell 7.2.3
Copyright (c) Microsoft Corporation.

https://aka.ms/powershell
Type 'help' to get help.

PS /> Connect-AzAccount -UseDeviceAuthentication
WARNING: To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code DR8XVZZDF to authenticate

Account                 SubscriptionName                     TenantId                             Environment
-------                 ----------------                     --------                             -----------
wim_wauters@outlook.com Azure subscription 1 (Pay As You Go) 30d3bf8c-20cd-4d59-99aa-95a27e09a326 AzureCloud
```

### Get users

Info can be found [here](https://docs.microsoft.com/en-us/powershell/module/az.resources/get-azaduser?view=azps-7.5.0)

```
PS /> Get-AzADUser -First 10

DisplayName   Id                                   Mail UserPrincipalName
-----------   --                                   ---- -----------------
Bjorn De Wulf bf28a5f6-9e1f-407a-9190-bd97e1c15065      bjorn@wimwautersoutlook.onmicrosoft.com
Wim Wauters   993cbc4c-144e-4970-8926-d9863f345faf      wim@azure.wimwauters.com
```

### Get Virtual Machines

```
PS /> Get-AzVM

ResourceGroupName          Name   Location          VmSize  OsType                              NIC ProvisioningState Zone
-----------------          ----   --------          ------  ------                              --- ----------------- ----
AZ-104                    wim-1 westeurope Standard_DS2_v2 Windows                         wim-1293         Succeeded
AZ-104                    wim-2 westeurope Standard_DS2_v2 Windows                          wim-288         Succeeded
EOC-RESOURCES       C8000-1-new westeurope Standard_DS2_v2   Linux {c8000-1-new140, c8000-1-new141}         Succeeded
EOC-RESOURCES       C8000-2-new westeurope Standard_DS2_v2   Linux {c8000-2-new628, c8000-2-new629}         Succeeded
EOC-RESOURCES     C8000v-OnPrem westeurope Standard_D2s_v3   Linux                 c8000v-onprem791         Succeeded
EOC-RESOURCES       C8000v-PA-1 westeurope Standard_D2s_v3   Linux    {c8000v-pa-1372, c8000v-pa-2}         Succeeded
EOC-RESOURCES       C8000v-PA-2 westeurope Standard_DS2_v2   Linux  {c8000v-pa-2265, C8000v-PA-2-1}         Succeeded
EOC-RESOURCES          ubuntu-1 westeurope Standard_D2s_v3   Linux                      ubuntu-1493         Succeeded
EOC-RESOURCES         ubuntu-pa westeurope Standard_D2s_v3   Linux                     ubuntu-pa722         Succeeded
EOC-RESOURCES        ubuntu-vpn westeurope Standard_D2s_v3   Linux                    ubuntu-vpn110         Succeeded
```


### Create new user

```
PS /> $secureStrPassword = ConvertTo-SecureString -String 'P@ssword' -AsPlainText -Force
PS /> New-AzADUser -DisplayName DemoUser -UserPrincipalName DemoUser@azure.wimwauters.com -Password $secureStrPassword -MailNickname DemoUser -ForceChangePasswordNextLogin

DisplayName Id                                   Mail UserPrincipalName
----------- --                                   ---- -----------------
DemoUser    056d4ce9-6261-44e8-a6a7-a917da192f57      DemoUser@azure.wimwauters.com
```


### Remove User

```
PS /> Get-AzADUser

DisplayName   Id                                   Mail UserPrincipalName
-----------   --                                   ---- -----------------
Bjorn De Wulf bf28a5f6-9e1f-407a-9190-bd97e1c15065      bjorn@wimwautersoutlook.onmicrosoft.com
DemoUser      056d4ce9-6261-44e8-a6a7-a917da192f57      DemoUser@azure.wimwauters.com
DemoWim       46d397d0-0994-457e-9da9-4ad051c4f18a      DemoWim@azure.wimwauters.com
Wim Wauters   993cbc4c-144e-4970-8926-d9863f345faf      wim@azure.wimwauters.com

PS /> Remove-AzADUser -ObjectId 056d4ce9-6261-44e8-a6a7-a917da192f57

PS /> Get-AzADUser

DisplayName   Id                                   Mail UserPrincipalName
-----------   --                                   ---- -----------------
Bjorn De Wulf bf28a5f6-9e1f-407a-9190-bd97e1c15065      bjorn@wimwautersoutlook.onmicrosoft.com
DemoWim       46d397d0-0994-457e-9da9-4ad051c4f18a      DemoWim@azure.wimwauters.com
Wim Wauters   993cbc4c-144e-4970-8926-d9863f345faf      wim@azure.wimwauters.com

PS /> Remove-AzADUser -DisplayName DemoWim
PS /> Get-AzADUser

DisplayName   Id                                   Mail UserPrincipalName
-----------   --                                   ---- -----------------
Bjorn De Wulf bf28a5f6-9e1f-407a-9190-bd97e1c15065      bjorn@wimwautersoutlook.onmicrosoft.com
Wim Wauters   993cbc4c-144e-4970-8926-d9863f345faf      wim@azure.wimwauters.com
```

