# Module 5 - Identity-Aware Microsegmentation

Calico eliminates the risks associated with lateral movement in the cluster to prevent access to sensitive data and other assets. Calico provides a unified, cloud-native segmentation model and single policy framework that works seamlessly across multiple application and workload environments. It enables faster response to security threats with a cloud-native architecture that can dynamically enforce security policy changes across cloud environments in milliseconds in response to an attack.

## Security Policy Tiers

Tiers are a hierarchical construct used to group policies and enforce higher precedence policies that other teams cannot circumvent, providing the basis for **Identity-aware microsegmentation**. 

All Calico and Kubernetes security policies reside in tiers. You can start “thinking in tiers” by grouping your teams and the types of policies within each group. The command below will create three tiers (quarantine, platform, and security):

```yaml
kubectl apply -f - <<-EOF   
apiVersion: projectcalico.org/v3
kind: Tier
metadata:
  name: security
spec:
  order: 300
---
apiVersion: projectcalico.org/v3
kind: Tier
metadata:
  name: platform
spec:
  order: 400
EOF
```

Policies are processed in sequential order from top to bottom.

![policy-processing](https://user-images.githubusercontent.com/104035488/206433417-0d186664-1514-41cc-80d2-17ed0d20a2f4.png)

Two mechanisms drive how traffic is processed across tiered policies:

- Labels and selectors
- Policy action rules

For more information about tiers, please refer to the Calico Cloud documentation [Understanding policy tiers](https://docs.calicocloud.io/get-started/tutorials/policy-tiers)

1. Implement explicitic policy to allow egress access from a workload in one namespace/pod, e.g. dev/centos, to default/frontend.
   
   a. Deploy egress policy between two namespaces dev and default.

   ```yaml
   kubectl apply -f - <<-EOF
   apiVersion: projectcalico.org/v3
   kind: NetworkPolicy
   metadata:
     name: platform.centos-to-frontend
     namespace: dev
   spec:
     tier: platform
     order: 100
     selector: app == "centos"
     egress:
       - action: Allow
         protocol: TCP
         source: {}
         destination:
           selector: app == "frontend"
           namespaceSelector: projectcalico.org/name == "default"
       - action: Pass
     types:
       - Egress
   EOF
   ```

   b. Test connectivity between dev/centos pod and default/frontend service again, should be allowed now.

   ```bash   
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://frontend.default 2>/dev/null | grep -i http'
   #output is HTTP/1.1 200 OK

2. Deploy the following security policy for allowing DNS access to all endpoints in the security ties.

  ```yaml
  kubectl apply -f - <<-EOF
  apiVersion: projectcalico.org/v3
  kind: GlobalNetworkPolicy
  metadata:
    name: security.allow-kube-dns
  spec:
    tier: security
    order: 200
    selector: all()
    types:
    - Egress    
    egress:
      - action: Allow
        protocol: UDP
        source: {}
        destination:
          selector: k8s-app == "kube-dns"
          ports:
          - '53'
      - action: Pass
  EOF
  ```

3. Apply the policies to allow the microservices to communicate with each other.

   ```bash
   kubectl apply -f msg
   ```

4. Use the Calico Cloud GUI to enforce the default-deny staged policy. After enforcing a staged policy, it takes effect immediatelly. The default-deny policy will start to actually deny traffic. 

---

## Security Policy Recommender

Notice that the Online Botique web site stop responding. That is because we didnt deploy any policy for the `frontend` worload.

Let's see how Calico can help us to build a security policy in order to allow the traffic to frontend service.

Click in the Policy Recommendation button in the Policy Board:

![recommend-a-policy](https://user-images.githubusercontent.com/104035488/206473864-f70ac0dd-d50d-46e1-b95b-d0153c154a79.png)

Now select the time range you will look back in the flow logs to recommend a policy based on them. Select the namespace of the application we want the recommended policy for (default), and the right service (frontend-XXXXXX-*).

When you click on the "Recommend" button in the top right corner, you will see that Calico recommends to open the traffic to port 8080 on Ingress, so we would be able to reach the frontend application again. There is also lots of other ingress and egress rules that will be created. Click on "Enforce", and then the "Back" button.
  
The policy will be created at the end of your policy chain (at the bottom of the default Tier). You must move the policy to the right order, so it can have effect. In our case, as we would like to hit this policy before the pci isolation policy is done (so we are able to reach the customer service before it is isolated), drag and drop the policy in the board to the right place as indicated by the figure below:

![move-policy](https://user-images.githubusercontent.com/104035488/206474337-cad7d5ef-da72-4845-97ee-8911e3249e62.png)

Now you should be able to access the Online Boutique application in your browser.

--- 

[:arrow_right: Module 6 - Identity-aware Microsegmentation](/modules/module-6-identity-aware-microsegmentation.md)  <br>

[:arrow_left: Module 4 - Workload Access Control](/modules/module-4-workload-access-control.md)  
[:leftwards_arrow_with_hook: Back to Main](/README.md)  
