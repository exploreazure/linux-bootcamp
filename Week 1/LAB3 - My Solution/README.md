# Lab 3: Manage Azure disks with the Azure CLI

1. Default Azure disks

https://docs.microsoft.com/en-us/azure/virtual-machines/linux/tutorial-manage-disks#default-azure-disks

Two types operating system disk and temporary disks (Note when a virtual machine is moved to another host, the data on the temp disk is lost)

2. Azure data disks

https://docs.microsoft.com/en-us/azure/virtual-machines/linux/tutorial-manage-disks#azure-data-disks



3. VM disk types

Standard (HDD), Ulta, Premimum and Standard (SDD)

https://docs.microsoft.com/en-us/azure/virtual-machines/disks-types#disk-type-comparison


4. Launch Azure Cloud Shell

https://shell.azure.com

5. Create and attach disks

```
    VMResourceGroup='rgLinuxServer01'
    VMName='linuxvm01'
    Subscription=$(az account show --query id --output tsv)

    az vm disk attach \
        --resource-group $VMResourceGroup \
        --vm-name $VMName \
        --name $VMName-disk1 \
        --size-gb 128 \
        --sku Standard_LRS \
        --new
```

6. Prepare data disks

```
    # from cloud shell
    ssh sysadmin@<publicipofvm>

    # once into the console
    # Create a partition
    sudo parted /dev/sdc --script mklabel gpt mkpart xfspart xfs 0% 100%

    # Write a file system to the partition
    sudo mkfs.xfs /dev/sdc1
    sudo partprobe /dev/sdc1

    # Mount Drive
    sudo mkdir /datadrive && sudo mount /dev/sdc1 /datadrive


```

7. Take a disk snapshot

```
    # Capture disk id
    osdiskid=$(az vm show \
   --resource-group $VMResourceGroup \
   --name $VMName \
   --query "storageProfile.osDisk.managedDisk.id" \
   -o tsv)

   # Capture a snapshot
   az snapshot create \
    --resource-group $VMResourceGroup \
    --source "$osdiskid" \
    --name osDisk-backup

   # Create a disk from snapshot
   az disk create \
   --resource-group $VMResourceGroup \
   --name mySnapshotDisk \
   --source osDisk-backup

   # Restore virtual machine from snapshot
   # delete vm, this doesn't delete disks
   az vm delete \
    --resource-group $VMResourceGroup \
    --name $VMName

   # Deploy vm from snapshot os disk
   az vm create \
    --resource-group $VMResourceGroup \
    --name $VMName \
    --attach-os-disk mySnapshotDisk \
    --os-type linux

   # Show data disks to be attached
   az disk list \
   --resource-group $VMResourceGroup \
   --query "[?contains(name,'$VMName')].[id]" \
   -o tsv

   az vm disk attach --vm-name $VMName --ids /subscriptions/$Subscription/resourceGroups/$VMResourceGroup/providers/Microsoft.Compute/disks/linuxvm01-disk1

   az vm disk attach --vm-name $VMName --ids /subscriptions/$Subscription/resourceGroups/$VMResourceGroup/providers/Microsoft.Compute/disks/linuxvm01_disk1_a0fd2a73aacc4ca2b05260c9bbc39d16

### Notes:

Quickstart: Manage Azure disks
* https://docs.microsoft.com/en-us/azure/virtual-machines/linux/tutorial-manage-disks

Quickstart for Bash in Azure Cloud Shell
* https://docs.microsoft.com/en-us/azure/cloud-shell/quickstart