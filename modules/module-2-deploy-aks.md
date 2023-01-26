# Azure AKS Cluster Creation - Azure CNI for Calico Cloud

1. Define the environment variables to be used by the resources definition.

   ```bash
   export RESOURCE_GROUP=zero-trust-workshop
   export CLUSTER_NAME=aks-workshop-cluster
   export LOCATION=canadacentral
   ```

2. If not created, create the Resource Group in the desired Region.
   
   ```bash
   az group create \
     --name $RESOURCE_GROUP \
     --location $LOCATION
   ```
   
3. Create the AKS cluster without a network plugin.
   
   ```bash
   az aks create \
     --resource-group $RESOURCE_GROUP \
     --name $CLUSTER_NAME \
     --kubernetes-version 1.23 \
     --location $LOCATION \
     --node-count 2 \
     --node-vm-size Standard_B2ms \
     --max-pods 100 \
     --generate-ssh-keys \
     --network-plugin azure
   ```

4. Get the credentials to connect to the cluster.
   
   ```bash
   az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
   ```
   
5. Verify the settings
   
   ```bash
   az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --query 'networkProfile'
   ```

   You should see "networkPlugin": "azure" and "networkPolicy": null (networkPolicy will just not show if it is null).

6. Verify the transparent mode by running the following command in one node

   ```bash
   VMSSGROUP=$(az vmss list | grep -i $RESOURCE_GROUP | awk -F ' ' '{print $2}')
   VMSSNAME=$(az vmss list | grep -i $RESOURCE_GROUP | awk -F ' ' '{print $1}')
   az vmss run-command invoke -g $VMSSGROUP -n $VMSSNAME --scripts "cat /etc/cni/net.d/*" --command-id RunShellScript --instance-id 0 --query 'value[0].message'
   ```
   
   > output should contain "mode": "transparent"


---

[:arrow_right: Module 3 - Deploy an AWS EKS cluster using Calico CNI](/modules/module-3-deploy-eks.md) <br>

[:arrow_left: Module 1 - Getting Started](/modules/module-1-getting-started.md)  
[:leftwards_arrow_with_hook: Back to Main](/README.md)  


[:leftwards_arrow_with_hook: Back to Main](/README.md#create-a-cluster-an-connect-it-to-calico-cloud)