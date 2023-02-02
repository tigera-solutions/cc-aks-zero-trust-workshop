# Module 2 - Deploy an Azure AKS Cluster

1. Define the environment variables to be used by the resources definition.

   > **NOTE**: In the following sections we'll be generating and setting some environment variables. If you're terminal session restarts you may need to reset these variables. You can use that via the following command:
   >
   > source ~/workshopvars.env

   ```bash
   export RESOURCE_GROUP=rg-zero-trust-workshop
   export CLUSTERNAME=aks-zero-trust-workshop
   export LOCATION=canadacentral
   # Persist for later sessions in case of disconnection.
   echo export RESOURCE_GROUP=$RESOURCE_GROUP >> ~/workshopvars.env
   echo export CLUSTERNAME=$CLUSTERNAME >> ~/workshopvars.env
   echo export LOCATION=$LOCATION >> ~/workshopvars.env
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
     --name $CLUSTERNAME \
     --kubernetes-version 1.23 \
     --location $LOCATION \
     --node-count 2 \
     --node-vm-size Standard_B2ms \
     --max-pods 100 \
     --generate-ssh-keys \
     --network-plugin azure
   ```

4. Verify your cluster status. The `ProvisioningState` should be `Succeeded`

   ```bash
   az aks list -o table
   ```
   Or
   ```bash
   watch az aks list -o table
   ```
   
   You may get an output like the following

   <pre>
   Name                     Location       ResourceGroup              KubernetesVersion    CurrentKubernetesVersion    ProvisioningState    Fqdn
   -----------------------  -------------  -------------------------  -------------------  --------------------------  -------------------  -----------------------------------------------------------------------
   aks-zero-trust-workshop  canadacentral  rg-zero-trust-workshop     1.23                 1.23.12                     Succeeded            aks-zero-t-rg-zero-trust-wo-03cfb8-b3feb0f8.hcp.canadacentral.azmk8s.io
   </pre>

5. Get the credentials to connect to the cluster.
   
   ```bash
   az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTERNAME
   ```

6. Verify you have API access to your new AKS cluster

   ```bash
   kubectl get nodes
   ```

   The output will ne something similar to the this:

   <pre>
   NAME                                STATUS   ROLES   AGE   VERSION
   aks-nodepool1-10832039-vmss000000   Ready    agent   18h   v1.23.12
   aks-nodepool1-10832039-vmss000002   Ready    agent   74m   v1.23.12
   </pre>

   To see more details about your cluster:

   ```bash
    kubectl cluster-info
   ```

   The output will ne something similar to the this:
   <pre>
   Kubernetes control plane is running at https://aks-zero-t-rg-zero-trust-wo-03cfb8-b3feb0f8.hcp.canadacentral.azmk8s.io:443
   CoreDNS is running at https://aks-zero-t-rg-zero-trust-wo-03cfb8-b3feb0f8.hcp.canadacentral.azmk8s.io:443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
   Metrics-server is running at https://aks-zero-t-rg-zero-trust-wo-03cfb8-b3feb0f8.hcp.canadacentral.azmk8s.io:443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

   To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
   </pre>

   You should now have a Kubernetes cluster running with 2 nodes. You do not see the master servers for the cluster because these are managed by Microsoft. The Control Plane services which manage the Kubernetes cluster such as scheduling, API access, configuration data store and object controllers are all provided as services to the nodes.

7. Verify the settings required for Calico Cloud.
   
   ```bash
   az aks show --resource-group $RESOURCE_GROUP --name $CLUSTERNAME --query 'networkProfile'
   ```

   You should see "networkPlugin": "azure" and "networkPolicy": null (networkPolicy will just not show if it is null).

8. Verify the transparent mode by running the following command in one node

   ```bash
   VMSSGROUP=$(az vmss list --output table | grep -i $RESOURCE_GROUP | awk -F ' ' '{print $2}')
   VMSSNAME=$(az vmss list --output table | grep -i $RESOURCE_GROUP | awk -F ' ' '{print $1}')
   az vmss run-command invoke -g $VMSSGROUP -n $VMSSNAME --scripts "cat /etc/cni/net.d/*" --command-id RunShellScript --instance-id 0 --query 'value[0].message' --output table
   ```
   
   > output should contain "mode": "transparent"

---

[:arrow_right: Module 3 - Connect the Azure AKS cluster to Calico Cloud](/modules/module-3-connect-calicocloud.md) <br>

[:arrow_left: Module 1 - Getting Started](/modules/module-1-getting-started.md)  
[:leftwards_arrow_with_hook: Back to Main](/README.md)  