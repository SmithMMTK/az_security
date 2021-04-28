# Initial variable

```bash
rg="00-az-sec2"
location="southeastasia"
## Workload Virtual Network
workload_vnet="workload-vnet"
workload_vnet_addressprefixes="10.1.0.0/16"
workload_subnet_snat_clusternodes="snet-workload"
workload_subnet_snat_clusternodes_prefix="10.1.0.0/24"

windowsVm="winVm"
ubuntuVm="ubuntuVm"
adminUser="azureuser"
adminPwd="Password123\!"

## Log Analytics Workspace
log="smi15sec210428955"

## Automation Account
autoAcct="smi15autoact"
```

# Create infratstructure
```bash
## Create Resource Group
az group create -n $rg -l $location

## Create Hub Virtual Network
az network vnet create \
    --name $workload_vnet \
    --address-prefixes $workload_vnet_addressprefixes \
    --resource-group $rg \
    --subnet-name $workload_subnet_snat_clusternodes \
    --subnet-prefixes $workload_subnet_snat_clusternodes_prefix

## Create Log Analytics workspace
az monitor log-analytics workspace create -g $rg -n $log
LOGID=$(az monitor log-analytics workspace show --resource-group $rg --workspace-name $log -o tsv --query "id")

## Create Windows VM
az vm create \
    --resource-group $rg \
    --name $windowsVm \
    --image win2016datacenter \
    --admin-username $adminUser \
    --admin-password $adminPwd \
    --vnet-name $workload_vnet \
    --subnet $workload_subnet_snat_clusternodes \
    --size Standard_D4s_v4 \
    --workspace $LOGID

## Create Ubuntu VM
az vm create \
    --resource-group $rg \
    --name $ubuntuVm \
    --image UbuntuLTS \
    --admin-username $adminUser \
    --admin-password $adminPwd \
    --size Standard_D4s_v4 \
    --vnet-name $workload_vnet --subnet $workload_subnet_snat_clusternodes \
    --workspace $LOGID

```

# Create Automation Account
```bash
az automation account create \
    -n $autoAcct \
    -g $rg \
    --sku "Basic"
```

To-do:
- Update management\Update management : Connect to Log Analytics workspace -> Enable
- Configuration Management\Inventory : Enable
- Add VM coverage (* Ensure VM was created with --workspace $LOGID)
    - Update management
    - Inventory
- Manage machines -> Enable on all avilable and future machines



