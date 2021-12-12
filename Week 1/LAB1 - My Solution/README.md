# Lab 1: Create a Linux virtual machine with the Azure CLI

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
    az vm list-sizes --location $Location

    VMName='linuxvm01'
    Image='UbuntuLTS'
    AdminUser='sysadmin'
    VMSize='Standard_B1s'

    az vm create \
    --resource-group $VMResourceGroup \
    --name $VMName \
    --size  $VMSize \
    --image $Image \
    --admin-username $AdminUser \
    --generate-ssh-keys

```

4. Open port 80 for web traffic

```
    HTTPPort=80
    az vm open-port --port $HTTPPort --resource-group $VMResourceGroup --name $VMName
```

5. Connect to virtual machine

```
    PublicIP=$(az vm list-ip-addresses --resource-group $VMResourceGroup --name $VMName --query "[].virtualMachine.network.publicIpAddresses[0].ipAddress" --output tsv)
    ssh $AdminUser@$PublicIP
```

6. Install web server

```
    sudo apt-get -y update
    sudo apt-get -y install nginx
```

7. View the web server in action

To test locally from within server:
In ssh to test type curl http://<publicip> - This should return a nginx test page in html format


To test externally:
Exit from ssh session, in cloud shell type http://<publicip>, this should also return the nginx test page

8, To Destory resources

```
    az group delete --name $VMResourceGroup
```