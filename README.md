# dapr-in-action

## Steps
The workshop is build around five steps.

0. Deployment of required Azure infrastructure, databases for step one and two, along with AKS for 3 and 4.
1. Building the monolithic Web API solution and splitting its databases.
2. Splitting solution into two projects, containeraizing them and adding DAPR runtime.
3. Create AKS manifest, setting up DAPR in Azure Kubernetes cluster and deploying solution to Cloud.
4. Adding DAPR pubsub component and RabbitMQ container. Changing solution code to work with a pubsub.
Analyzing issues and solving them with logging.


## Prerequisites

Good mood :).

## Step 0. Azure infrastructure
Script below should be run via Azure Portal bash console. 
You will receive database connection strings with setx command as output of this script.
Please add a correct name of your subscription to the first row of the script. 

```bash
subscriptionID=$(az account list --query "[?contains(name,'Microsoft')].[id]" -o tsv)
echo "Test subscription ID is = " $subscriptionID
az account set --subscription $subscriptionID
az account show

location=northeurope
postfix=$RANDOM

#----------------------------------------------------------------------------------
# Database infrastructure
#----------------------------------------------------------------------------------

export dbResourceGroup=net-fwdays-dapr-data$postfix
export dbServername=net-fwdays-dapr$postfix
export dbPoolname=fwdays
export dbAdminlogin=FancyUser3
export dbAdminpassword=Sup3rStr0ng52$postfix
export dbPaperName=paperorders
export dbDeliveryName=deliveries

az group create --name $dbResourceGroup --location $location

az sql server create --resource-group $dbResourceGroup --name $dbServername --location $location \
--admin-user $dbAdminlogin --admin-password $dbAdminpassword
	
az sql elastic-pool create --resource-group $dbResourceGroup --server $dbServername --name $dbPoolname \
--edition Standard --dtu 50 --zone-redundant false --db-dtu-max 50

az sql db create --resource-group $dbResourceGroup --server $dbServername --elastic-pool $dbPoolname \
--name $dbPaperName --catalog-collation SQL_Latin1_General_CP1_CI_AS
	
az sql db create --resource-group $dbResourceGroup --server $dbServername --elastic-pool $dbPoolname \
--name $dbDeliveryName --catalog-collation SQL_Latin1_General_CP1_CI_AS	

sqlClientType=ado.net

SqlPaperString=$(az sql db show-connection-string --name $dbPaperName --server $dbServername --client $sqlClientType --output tsv)
SqlPaperString=${SqlPaperString/Password=<password>;}
SqlPaperString=${SqlPaperString/<username>/$dbAdminlogin}

SqlDeliveryString=$(az sql db show-connection-string --name $dbDeliveryName --server $dbServername --client $sqlClientType --output tsv)
SqlDeliveryString=${SqlDeliveryString/Password=<password>;}
SqlDeliveryString=${SqlDeliveryString/<username>/$dbAdminlogin}

SqlPaperPassword=$dbAdminpassword

#----------------------------------------------------------------------------------
# AKS infrastructure
#----------------------------------------------------------------------------------

location=northeurope
groupName=netfwdays-cluster$postfix
clusterName=netfwdays-cluster$postfix
registryName=netfwdaysregistry$postfix
accountSku=Standard_LRS
accountName=netfwdaysstorage$postfix
queueName=netfwdaysqueue
queueResultsName=netfwdaysqueueresults

az group create --name $groupName --location $location

az storage account create --name $accountName --location $location --kind StorageV2 \
--resource-group $groupName --sku $accountSku --access-tier Hot  --https-only true

accountKey=$(az storage account keys list --resource-group $groupName --account-name $accountName --query "[0].value" | tr -d '"')

accountConnString="DefaultEndpointsProtocol=https;AccountName=$accountName;AccountKey=$accountKey;EndpointSuffix=core.windows.net"

az storage queue create --name $queueName --account-key $accountKey \
--account-name $accountName --connection-string $accountConnString

az storage queue create --name $queueResultsName --account-key $accountKey \
--account-name $accountName --connection-string $accountConnString

az acr create --resource-group $groupName --name $registryName --sku Standard
az acr identity assign --identities [system] --name $registryName

az aks create --resource-group $groupName --name $clusterName --node-count 3 --generate-ssh-keys --network-plugin azure
az aks update --resource-group $groupName --name $clusterName --attach-acr $registryName

echo "Update local.settings.json Values=>AzureWebJobsStorage value with:  " $accountConnString

#----------------------------------------------------------------------------------
# SQL connection strings
#----------------------------------------------------------------------------------

printf "\n\nRun string below in local cmd prompt to assign secret to environment variable SqlPaperString:\nsetx SqlPaperString \"$SqlPaperString\"\n\n"
printf "\n\nRun string below in local cmd prompt to assign secret to environment variable SqlDeliveryString:\nsetx SqlDeliveryString \"$SqlDeliveryString\"\n\n"
printf "\n\nRun string below in local cmd prompt to assign secret to environment variable SqlPaperPassword:\nsetx SqlPaperPassword \"$SqlPaperPassword\"\n\n"

```

2. Splitting solution into two projects, containeraizing them and adding DAPR runtime.
3. Create AKS manifest, setting up DAPR in Azure Kubernetes cluster and deploying solution to Cloud.
4. Adding DAPR pubsub component and RabbitMQ container. Changing solution code to work with a pubsub.

## Step 1. Monolithic Web API initial split.

The idea of this step is to start with monolith split without breaking it into several solutions.
First we will split Entity Framework context into two parts that can use the same(of different databases)

## Step 1. Monolithic Web API initial split.

## Step 2. Split in two projects, docker compose and DAPR initialization.
We adding containerization via Visual Studio tooling and manually adding DAPR sidecar configuration for each server.

## Step 3. Application deployment to Azure Kubernetes service.
We will create AKS manifests for our services and add DAPR sections.
Deploy dapr to AKS cluster and add containers to the private repository.

## Step 4. Introduction to the DAPR pubsub.
We will deploy DAPR pubsub component to Azure. Make changes to our code and take a look into the pod logs to see whats happening.


