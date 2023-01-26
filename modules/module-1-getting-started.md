# Getting Started

## Prerequisites 

The following are the basic requirements to **start** the labs. Individual labs may have other requirements that will be listed within the lab.

* Azure Account [Azure Portal](https://portal.azure.com)
* Git [Git SCM](https://git-scm.com/downloads)
* Azure Cloud Shell [https://shell.azure.com](https://shell.azure.com)

## Application

# Lab 1: Create AKS Cluster

In this lab we will create our Azure Kubernetes Services (AKS) distributed compute cluster.

## Prerequisites

- Azure Account

## Instructions

1. Login to Azure Portal at http://portal.azure.com.
2. Open the Azure Cloud Shell and choose Bash Shell (do not choose Powershell)

   ![Azure Cloud Shell](img-cloud-shell.png "Azure Cloud Shell")

3. The first time Cloud Shell is started will require you to create a storage account.

4. Once your cloud shell is started, clone the workshop repo into the cloud shell environment

   ```bash
   git clone https://github.com/Azure/kubernetes-hackfest
   ```

   > Note: In the cloud shell, you are automatically logged into your Azure subscription.

5. Ensure you are using the correct Azure subscription you want to deploy AKS to.

   ```
   # View subscriptions
   az account list
   ```

   ```
   # Verify selected subscription
   az account show
   ```

   ```
   # Set correct subscription (if needed)
   az account set --subscription <subscription_id>

   # Verify correct subscription is now set
   az account show
   ```

6. Create a unique identifier suffix for resources to be created in this lab.

   > *NOTE:* In the following sections we'll be generating and setting some environment variables. If you're terminal session restarts you may need to reset these variables. You can use that via the following command:
   >
   > source ~/workshopvars.env

   ```bash
   echo "# Start AKS Hackfest Lab Params">>~/workshopvars.env
   UNIQUE_SUFFIX=$USER$RANDOM
   # Remove Underscores and Dashes (Not Allowed in AKS and ACR Names)
   UNIQUE_SUFFIX="${UNIQUE_SUFFIX//_}"
   UNIQUE_SUFFIX="${UNIQUE_SUFFIX//-}"
   # Check Unique Suffix Value (Should be No Underscores or Dashes)
   echo $UNIQUE_SUFFIX
   # Persist for Later Sessions in Case of Timeout
   echo export UNIQUE_SUFFIX=$UNIQUE_SUFFIX >> ~/workshopvars.env
   ```

7. Create an Azure Resource Group in East US.

   ```bash
   # Set Resource Group Name using the unique suffix
   RGNAME=aks-rg-$UNIQUE_SUFFIX
   # Persist for Later Sessions in Case of Timeout
   echo export RGNAME=$RGNAME >> ~/workshopvars.env
   # Set Region (Location)
   LOCATION=eastus
   # Persist for Later Sessions in Case of Timeout
   echo export LOCATION=eastus >> ~/workshopvars.env
   # Create Resource Group
   az group create -n $RGNAME -l $LOCATION
   ```

8. Create your AKS cluster in the resource group created above with 3 nodes. We will check for a recent version of kubnernetes before proceeding. We are also including the monitoring add-on for Azure Container Insights. You will use the Service Principal information from step 5.

   Use Unique CLUSTERNAME

   ```bash
   # Set AKS Cluster Name
   CLUSTERNAME=aks${UNIQUE_SUFFIX}
   # Look at AKS Cluster Name for Future Reference
   echo $CLUSTERNAME
   # Persist for Later Sessions in Case of Timeout
   echo export CLUSTERNAME=aks${UNIQUE_SUFFIX} >> ~/workshopvars.env
   ```

   Get available kubernetes versions for the region. You will likely see more recent versions in your lab.

   ```bash
   az aks get-versions -l $LOCATION --output table

   KubernetesVersion    Upgrades
   -------------------  ----------------------
   1.23.5               None available
   1.23.3               1.23.5
   1.22.6               1.23.3, 1.23.5
   1.22.4               1.22.6, 1.23.3, 1.23.5
   1.21.9               1.22.4, 1.22.6
   1.21.7               1.21.9, 1.22.4, 1.22.6
   ```

   For this lab we'll use 1.23.5

   ```bash
   K8SVERSION=1.23.5
   ```

   > The below command can take 3-5 minutes to run as it is creating the AKS cluster. 

   ```bash
   # Create AKS Cluster
   az aks create -n $CLUSTERNAME -g $RGNAME \
   --kubernetes-version $K8SVERSION \
   --enable-managed-identity \
   --generate-ssh-keys -l $LOCATION \
   --node-count 3 \
   --no-wait
   ```

9.  Verify your cluster status. The `ProvisioningState` should be `Succeeded`

   ```bash
   az aks list -o table
   # Or
   watch az aks list -o table
   ```

   ```bash
      Name            Location    ResourceGroup      KubernetesVersion    ProvisioningState    Fqdn
      --------------  ----------  -----------------  -------------------  -------------------  ----------------------------------------------------------------
      aks25415        eastus      aks-rg-25415       1.23.5               Succeeded            aks25415-aks-rg-25415-62afe9-3a0152d0.hcp.eastus.azmk8s.io
   ```

10.  Get the Kubernetes config files for your new AKS cluster

   ```bash
     az aks get-credentials -n $CLUSTERNAME -g $RGNAME
   ```

12.  Verify you have API access to your new AKS cluster

   ```bash
   kubectl get nodes
   ```

   ```bash
   NAME                                STATUS   ROLES   AGE     VERSION
   aks-nodepool1-25335207-vmss000000   Ready    agent   2m1s    v1.23.5
   aks-nodepool1-25335207-vmss000001   Ready    agent   2m5s    v1.23.5
   aks-nodepool1-25335207-vmss000002   Ready    agent   2m13s   v1.23.5
   ```

   To see more details about your cluster:

   ```bash
    kubectl cluster-info
   
   Kubernetes control plane is running at https://akssteve10-aks-rg-steve1075-62afe9-631a3ab4.hcp.eastus.azmk8s.io:443
   CoreDNS is running at https://akssteve10-aks-rg-steve1075-62afe9-631a3ab4.hcp.eastus.azmk8s.io:443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
   Metrics-server is running at https://akssteve10-aks-rg-steve1075-62afe9-631a3ab4.hcp.eastus.azmk8s.io:443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

   To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
   ```

    You should now have a Kubernetes cluster running with 3 nodes. You do not see the master servers for the cluster because these are managed by Microsoft. The Control Plane services which manage the Kubernetes cluster such as scheduling, API access, configuration data store and object controllers are all provided as services to the nodes.