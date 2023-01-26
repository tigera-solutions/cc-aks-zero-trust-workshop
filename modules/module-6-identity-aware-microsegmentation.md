
## Microsegmentation

Calico eliminates the risks associated with lateral movement in the cluster to prevent access to sensitive data and other assets. Calico provides a unified, cloud-native segmentation model and single policy framework that works seamlessly across multiple application and workload environments. It enables faster response to security threats
with a cloud-native architecture that can dynamically enforce security policy changes across cloud environments in milliseconds in response to an attack.


| PCI Control # | Requirements| How Calico meets this requirements |
| --- | --- | --- |
|1.1, 1.1.4, 1.1.6, 1.2.1, 1.2.2, 1.2.3 | Install and maintain a firewall configuration to protect cardholder data | • Identify everything covered by PCI requirements with a well-defined label (e.g. PCI=true)<br>• Block all traffic between PCI and non-PCI workloads<br>• Whitelist all traffic within PCI workloads |

 
### Microsegmentation using label PCI = true on a namespace

1. For the microsegmentation deploy a new example application

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/regismartins/cc-aks-security-compliance-workshop/main/manifests/storefront-pci.yaml
   ```

2. Verify that all the workloads has the label `PCI=true`.

   ```bash
   kubectl get pods -n storefront --show-labels
   ```

3. Create a policy that only allows endpoints with label PCI=true to communicate.

   ```yaml
   kubectl apply -f - <<-EOF
   apiVersion: projectcalico.org/v3
   kind: GlobalNetworkPolicy
   metadata:
     name: security.pci-whitelist
   spec:
     tier: security
     order: 100
     selector: projectcalico.org/namespace == "storefront"
     ingress:
     - action: Deny
       source:
         selector: PCI != "true"
       destination:
         selector: PCI == "true"
     - action: Pass
       source:
       destination:
     egress:
     - action: Allow
       protocol: UDP
       source: {}
       destination:
         selector: k8s-app == "kube-dns"
         ports:
         - '53'
     - action: Deny
       source:
         selector: PCI == "true"
       destination:
         selector: PCI != "true"
     - action: Pass
       source:
       destination:
     types:
     - Ingress
     - Egress
   EOF
   ```

Now only the pods labeled with PCI=true will be able to exchange information. Note that you can use different labels to create any sort of restrictions for the workloads communications.
