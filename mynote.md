az login

// Resource group
az group create --name migroup --location westus2

// AzurePostGres DB
az postgres server create --resource-group migroup --name mipostgresdb --location westus2 --admin-user myadmin --admin-password Test@123456 --sku-name B_Gen5_1
// Configure firewall rules to allow all IPs to connect:
az postgres server firewall-rule create --resource-group migroup --server mipostgresdb --name AllowAllIPs --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255

// Create a new database techconfdb:
az postgres db create --resource-group migroup --server-name mipostgresdb --name techconfdb

// Restore the database with the backup located in the data folder:
PGPASSWORD=Test@123456 psql --host=mipostgresdb.postgres.database.azure.com --port=5432 --username=myadmin@mipostgresdb --dbname=techconfdb --file=data/techconfdb_backup.sql

// Create a Service Bus namespace:
az servicebus namespace create --resource-group migroup --name miservicebusns --location westus2

// Create a Service Bus queue named notificationqueue:
az servicebus queue create --resource-group migroup --namespace-name miservicebusns --name notificationqueue

// Get the Service Bus connection string:
SERVICE_BUS_CONNECTION_STRING=$(az servicebus namespace authorization-rule keys list --resource-group migroup --namespace-name myservicebusnamespace --name RootManageSharedAccessKey --query primaryConnectionString --output tsv)

// Create app service plan
az appservice plan create --name miAppSvcPlan --resource-group migroup --location westus2 --sku B1

// Create a storage account
az storage account create --name migroupstorageacc --resource-group migroup --location westus2 --sku Standard_LRS

### Deploy the web app
// Create web app
az webapp create --resource-group migroup --plan miAppSvcPlan --name migroupwebapp --runtime "PYTHON|3.8"

// Deploy your code to the Web App
az webapp deployment source config-local-git --name migroupwebapp --resource-group migroup

