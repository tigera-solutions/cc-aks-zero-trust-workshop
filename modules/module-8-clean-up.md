7. Clean up

   Delete the AKS cluster
   
   ```bash
   az aks delete \
     --resource-group $RESOURCE_GROUP \
     --name $CLUSTER_NAME
   ```

   Delete the resource group
   
   ```bash
   az group delete \
     --name $RESOURCE_GROUP \
     --location $LOCATION
   ```