# Play.Catalog
Play Economy Play.Catalog microservice.

## Create and publish package
```powershell
$version="1.0.4"
$packageversion="1.0.3"
$owner="jeevdotnetmicroservice"
$gh_pat="[PAT HERE]"

dotnet pack src\Play.Catalog.Contracts\ --configuration Release -p:PackageVersion=$packageversion -p:RepositoryUrl=https://github.com/$owner/Play.Catalog -o ..\packages

dotnet nuget push ..\packages\Play.Catalog.Contracts.$packageversion.nupkg --api-key $gh_pat --source "github"
```

## Build the docker image
```powershell
$rpname="jeevplayeconomy"
$env:GH_OWNER="jeevdotnetmicroservice"
$env:GH_PAT="[PAT HERE]"
docker build --secret id=GH_OWNER --secret id=GH_PAT -t "$rpname.azurecr.io/play.catalog:$version" .
```

## Run the docker image
```powershell
$cosmosDbConnString="[CONN STRING HERE]"
$serviceBusConnString="[CONN STRING HERE]"

docker run -it --rm -p 5000:5000 --name catalog -e MongoDbSettings__ConnectionString=$cosmosDbConnString -e ServiceBusSettings__ConnectionString=$serviceBusConnString -e ServiceSettings__MessageBroker="SERVICEBUS" play.catalog:$version
```

## Publishing the Docker image
```powershell
az acr login --name $rpname
docker push "$rpname.azurecr.io/play.catalog:$version" 
```

### Creating the pod managed identity
```powershell
$rgname="playeconomy"
$namespace="catalog"
az identity create --resource-group $rgname --name $namespace
$IDENTITY_RESOURCE_ID=az identity show -g $rgname -n $namespace --query id -otsv

az aks pod-identity add --resource-group $rgname --cluster-name $rpname --namespace $namespace --name $namespace --identity-resource-id $IDENTITY_RESOURCE_ID
```

## Granting access to Key Vault secrets
```powershell
$IDENTITY_CLIENT_ID=az identity show -g $rgname -n $namespace --query clientId -otsv
az keyvault set-policy -n $rpname --secret-permissions get list --spn $IDENTITY_CLIENT_ID
```