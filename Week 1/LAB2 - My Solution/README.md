# Lab 2: Create a Linux virtual machine with the Azure CLI


## Introduction
In this lab from the Cloud skills Linux bootcamp we deploy a Liunx virtual machine and use the custom script extension to install the Nginx web server.

This lab uses a user assigned identity on both the virtual machine and the storage account.


1. Launch Azure Cloud Shell

https://shell.azure.com

Select Bash

2. Create a resource group

```
    VMResourceGroup='rgLinuxServer01'
    Location='northeurope'
    az group create --name $VMResourceGroup --location $Location
```

3. Create virtual machine

```
    # Get available vmsizes in region to help you decide best vmsize
    az vm list-sizes --location $Location --output table

    VMName='linuxvm01'
    Image='UbuntuLTS'
    AdminUser='sysadmin'
    VMSize='Standard_B1s'

    # Note below command creates a linux virtual machine

    az vm create \
    --resource-group $VMResourceGroup \
    --name $VMName \
    --size  $VMSize \
    --image $Image \
    --admin-username $AdminUser \
    --generate-ssh-keys


```

4. Understand VM images

```
   # Find all Ubuntu images
   az vm image list --all --publisher Canonical
```

5. Understand VM sizes

```
    # Get available vmsizes for virtual machine
    az vm list-vm-resize-options \
    --resource-group $VMResourceGroup \
    --name $VMName --output table

    NewVMSize='Standard_B2s'
    # Resize virtual machine
    az vm resize \
    --resource-group $VMResourceGroup \
    --name $VMName \
    --size $NewVMSize
```

6. VM power states

```
    # To power off virtual machine
    az vm deallocate \
    --resource-group $VMResourceGroup \
    --name $VMName

    # To power on virtual machine
    az vm start \
    --resource-group $VMResourceGroup \
    --name $VMName
```

7. User Assigned Identity


```

    userAssignedIdentity='lab2UserId'

    # Create user assigned identity
    az identity create --resource-group $VMResourceGroup --name $userAssignedIdentity

    # Assigned to virtual machine
    az vm identity assign --resource-group $VMResourceGroup --name $VMName --identities $userAssignedIdentity   

    # To verify user assigned identity, use following command
    az vm identity show --name $VMName --resource-group $VMResourceGroup

    # the output of the above command, shows the client id. Add this value to the below variable
    clientId='<clientid>'

```

8. Management tasks

```
    StorageAccountName='salinuxserv6789'
    StorageAccountSKU='Standard_LRS'
    StorageAccountKind='StorageV2'
    StorageAccountContainer='scripts'
    ScriptName='InstallNginx.sh'
    ScriptFile='InstallNginx.sh'
    Subscription=$(az account show --query id --output tsv)
  

    az storage account create \
    --name $StorageAccountName \
    --resource-group $VMResourceGroup \
    --location $Location \
    --sku $StorageAccountSKU \
    --kind StorageV2

    # create this file in cloud shell using code InstallNginx.sh

    #!/bin/sh
    # This script installs nginx web server
    sudo apt-get -y update
    sudo apt-get -y install nginx


    # Create assignment to allow user to upload script
    az ad signed-in-user show --query objectId -o tsv | az role assignment create \
    --role "Storage Blob Data Contributor" \
    --assignee @- \
    --scope "/subscriptions/$Subscription/resourceGroups/$VMResourceGroup/providers/Microsoft.Storage/storageAccounts/$StorageAccountName"

    # create container in storage account
  
    az storage container create \
    --account-name $StorageAccountName \
    --name $StorageAccountContainer \
    --auth-mode login

    # upload script, note this must be uploading into cloud shell prior to uploading into blob storage
    
    az storage blob upload \
    --account-name $StorageAccountName \
    --container-name $StorageAccountContainer \
    --name $ScriptName \
    --file $ScriptFile \
    --auth-mode login

    # Create a reader RBAC assignment, to allow the VM user assigned identity attached to read storage account
    az role assignment create \
    --role "Storage Blob Data Reader" \
    --assignee $clientId \
    --scope "/subscriptions/$Subscription/resourceGroups/$VMResourceGroup/providers/Microsoft.Storage/storageAccounts/$StorageAccountName"

    # Open Port 80
    az vm open-port --port 80 --resource-group $VMResourceGroup --name $VMName


    # I created a JSON file called customscript.json. We can apply this JSON to VM to run the script against VM
    # Upload JSON into cloudshell
    az deployment group create --resource-group $VMResourceGroup --template-file "customscript.json"   
    # Note the first time this command is run, the custom script extension is installed. Therefore, the script will initially take some time to execute
```


This has succesfully installed Nginx, without needing to sign into the ssh console.
To test http://<publicip>
The Nginx test web page should be shown


9. To Destory resources

```
    az group delete --name $VMResourceGroup
```