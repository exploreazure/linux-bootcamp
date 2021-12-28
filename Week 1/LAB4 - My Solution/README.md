# Lab 4: Create and use an SSH public-private key pair for Linux VMs in Azure

1. Supported SSH key formats

Azure currently supports SSH protocol 2 (SSH-2) RSA public-private key pairs with a minimum length of 2048 bits. Other key formats such as ED25519 and ECDSA are not supported.

2. Create an SSH key pair

```
ssh-keygen -m PEM -t rsa -b 4096 -f ~/.ssh/lab4key
```

Note specify a file, otherwise the default files will be overwritten


3. Provide an SSH public key when deploying a VM

```
    VMName='linuxvm01'
    Image='UbuntuLTS'
    AdminUser='sysadmin'
    VMSize='Standard_B1s'
    VMResourceGroup='rgLinuxServer01'

    az vm create \
    --resource-group $VMResourceGroup \
    --name $VMName \
    --size  $VMSize \
    --image $Image \
    --admin-username $AdminUser \
    --ssh-key-value ~/.ssh/lab4key.pub
```

4. SSH into your VM

```
ssh -i ~/ssh/lab4key sysadmin@<publicip>
```

### Notes:

Quickstart: SSH for Linux VMs
* https://docs.microsoft.com/en-us/azure/virtual-machines/linux/mac-create-ssh-keys

Quickstart for Bash in Azure Cloud Shell
* https://docs.microsoft.com/en-us/azure/cloud-shell/quickstart