
# Module 1 - Connect your AKS cluster to Calico Cloud

> [!IMPORTANT]
> To complete this module, you will need a [Calico Cloud account](https://www.calicocloud.io/). If you are participating in a live workshop, you will receive an invite with the information to login into an active Calico Cloud environment. Once logged in,  join your AKS cluster there.
If you are running this workshop in a self-paced mode, you can create a Calico Cloud environment following the steps [here](/modules/submodule-1.1-create-calicloud.md).  

Issues with being unable to navigate menus in the UI are often due to browsers blocking scripts - please ensure that you disable all blocker scripts.

## Step 1 - Accept the Invitation

1. During the workshop, you will receive an invitation to connect to a Calico Cloud organization, similar to the picture below:

   ![accept_invitation](https://user-images.githubusercontent.com/104035488/215204989-66b666d9-5e93-45b5-a0c5-2236b135af31.png)

2. Click on the link ACCEPT INVITATION and create a password to access the Calico Cloud.

   <img width="300" alt="create a password" src="https://user-images.githubusercontent.com/104035488/215205017-0a41f506-5c91-4830-9249-677c6a06fb3b.png">

3. Once you have access to your **Calico Cloud** environment, go to step 2:

## Step 2 - Connect your cluster to Calico Cloud

1. The welcome screen will guide you to the process of connecting your cluster to Calico Cloud. Let's proceed and connect our first cluster on the **Managed Clusters** section. Click on the "**Connect Cluster**" button to start the process.

   The Connect Cluster window will allow you to choose a name to identify your cluster in Calico Cloud and select which platform you are running the cluster on. The next window presents a link for you to review the cluster requirements for Calico Cloud. Calico Cloud will generate a kubectl command to run the installation script, and you need to copy and apply this command in your cluster.

   > **:warning: Attention**: During the live workshop, the Calico Cloud cluster will be shared. To avoid confusion, you can create a random prefix for your cluster name. This name can be achieved using the following command on the Cloud9 prompt:

    ```bash
    echo $RANDOM-tigera-workshop
    ```

   ![registering_get_key](https://user-images.githubusercontent.com/104035488/188036064-f85cac4f-66c0-4c09-bdd3-67922640679d.gif)

2. Run the installation script in your cluster. Script should look similar to this:

    <pre>
    kubectl apply -f https://installer.calicocloud.io/manifests/cc-operator/lat
    est/deploy.yaml && curl -H "Authorization: Bearer a7c2oex34:00llxrhcq:1ga2c
    z69d7ug81yjgakpyclv6o3eu8o97kp7t2483lmwajslu47xed94e4ic8ywn" "https://www.c
    alicocloud.io/api/managed-cluster/deploy.yaml" | kubectl apply -f -
    </pre>

    > Output should look similar to:
    <pre>
    namespace/calico-cloud created
    customresourcedefinition.apiextensions.k8s.io/installers.operator.calicocloud.io created
    serviceaccount/calico-cloud-controller-manager created
    role.rbac.authorization.k8s.io/calico-cloud-leader-election-role created
    clusterrole.rbac.authorization.k8s.io/calico-cloud-metrics-reader created
    clusterrole.rbac.authorization.k8s.io/calico-cloud-proxy-role created
    rolebinding.rbac.authorization.k8s.io/calico-cloud-leader-election-rolebinding created
    clusterrolebinding.rbac.authorization.k8s.io/calico-cloud-installer-rbac created
    clusterrolebinding.rbac.authorization.k8s.io/calico-cloud-proxy-rolebinding created
    configmap/calico-cloud-manager-config created
    service/calico-cloud-controller-manager-metrics-service created
    deployment.apps/calico-cloud-controller-manager created
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                    Dload  Upload   Total   Spent    Left  Speed
    100   355  100   355    0     0    541      0 --:--:-- --:--:-- --:--:--   541
    secret/api-key created
    installer.operator.calicocloud.io/aks-cc-repo created
    </pre>

    Joining the cluster to Calico Cloud can take a few minutes. Meanwhile the Calico resources can be monitored until they are all reporting `Available` as `True`.

    ```bash
    kubectl get tigerastatus                                    
    ```

    > Output should look similar to:
    <pre>
    NAME                            AVAILABLE   PROGRESSING   DEGRADED   SINCE
    apiserver                       True        False         False      96s
    calico                          True        False         False      16s
    compliance                      True        False         False      21s
    intrusion-detection             True        False         False      41s
    log-collector                   True        False         False      21s
    management-cluster-connection   True        False         False      51s
    monitor                         True        False         False      2m1s
    </pre>

    You can also monitor your cluster installation on the Calico Cloud UI. Go to the "**Managed Clusters**" section, select your cluster and expand the timestamp dropdown to see the installation logs.
    In a few minutes, the status will change from **Installing** to **Done**. Congratulations! You successfully connected your cluster to Calico Cloud.

    ![installing](https://user-images.githubusercontent.com/104035488/188036070-71cd3cb7-639b-46f2-bd5e-dbdb401b48e3.gif)

## STEP 3 - Select your cluster

Once the installation is completed, you can start interacting with your cluster from the Calico Cloud interface. Calico Cloud provides a single pane of glass for managing multiple clusters. If you followed the previous steps, you would have two clusters connected to Calico Cloud at this point: Your cluster and a pre-configured lab cluster that allows you to explore some of the features in Calico Cloud.

You can switch between clusters by following the steps below:

1. Navigate to the Dashboard section - the first icon under the Calico Cat on the top-left of the UI.

2. Click on the **Cluster** dropdown button on the top-right of the UI.

3. Select your recently added cluster.

   ![selecting_cluster](https://user-images.githubusercontent.com/104035488/188036074-857e6a19-7641-4dff-9f6b-02eb627cf748.gif)

The "**Cluster**" dropdown button will always be visible across the Calico Cloud UI, no matter which section you are viewing. You can change the cluster you want to interact with at any moment.  
When you change the cluster, the Calico Cloud context will change immediately to reflect the information regarding the currently selected cluster.

---

[:arrow_right: Module 2 - Zero-Trust Workload Access Control with Namespace Isolation Recommendation](/mod/module-2-ztac-ns-isolation.md)   <br>

[:arrow_left: Back to Main](/README.md)  
