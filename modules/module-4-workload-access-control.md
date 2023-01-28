# Module 4 - Zero Trust Workload Access Control

Calico provides methods to enable fine-grained access controls between your microservices and external databases, cloud services, APIs, and other applications that are protected behind a firewall. You can enforce controls from within the cluster using DNS egress policies, from a firewall outside the cluster using the egress gateway. Controls are applied on a fine-grained, per-pod basis.

In this module, we will learn how to use Calico to create network policies to control access to and from a pod. For this, we will install two example application stacks: The Online Boutique and the Dev environment. Once the applications are deployed, we will create and test network security policies with different ingress and egress rules to demonstrate how the **workload access control** is done.

1. Installing the example application stacks:

   From the cloned directory, execute:
   ```
   kubectl apply -f pre
   ```

   > **Note**: Wait until all the pods are up and running to move to the next step.

## Service Graph and Flow Visualizer

Connect to Calico Cloud GUI. From the menu select `Service Graph > Default`. Explore the options.

![service_graph](https://user-images.githubusercontent.com/104035488/192303379-efb43faa-1e71-41f2-9c54-c9b7f0538b34.gif)

Connect to Calico Cloud GUI. From the menu select `Service Graph > Flow Visualizations`. Explore the options.

![flow-visualization](https://user-images.githubusercontent.com/104035488/192358472-112c832f-2fd7-4294-b8cc-fec166a9b11e.gif)

## Network Security Policies

Calico Security Policies provide a richer set of policy capabilities than Kubernetes network policies including:  

- Policies that can be applied to any kind of endpoint: pods/containers, VMs, and/or to host interfaces
- Policies that can define rules that apply to ingress, egress, or both
- Policy rules support:
  - Actions: allow, deny, log, pass
  - Source and destination match criteria:
    - Ports: numbered, ports in a range, and Kubernetes named ports
    - Protocols: TCP, UDP, ICMP, SCTP, UDPlite, ICMPv6, protocol numbers (1-255)
    - HTTP attributes (if using Istio service mesh)
    - ICMP attributes
    - IP version (IPv4, IPv6)
    - IP or CIDR
    - Endpoint selectors (using label expression to select pods, VMs, host interfaces, and/or network sets)
    - Namespace selectors
    - Service account selectors

### The Zero Trust approach

A global default deny policy ensures that unwanted traffic (ingress and egress) is denied by default. Pods without policy (or incorrect policy) are not allowed traffic until appropriate network policy is defined. Although the staging policy tool will help you find incorrect and missing policy, a global deny helps mitigate against other lateral malicious attacks.

By default, all traffic is allowed between the pods in a cluster. Let's start by testing connectivity between application components and across application stacks. All of these tests should succeed as there are no policies in place.

First, let's install `curl` in the loadgenerator pod for these tests.

```bash
kubectl exec -it $(kubectl get po -l app=loadgenerator -ojsonpath='{.items[0].metadata.name}') -c main -- sh -c 'apt-get update && apt install curl -y'
```

a. Test connectivity between workloads within each namespace, use dev and default namespaces as example

   ```bash
   # test connectivity within dev namespace, the expected result is "HTTP/1.1 200 OK" 
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://nginx-svc 2>/dev/null | grep -i http'
   ```

   ```bash
   # test connectivity within default namespace in 8080 port
   kubectl exec -it $(kubectl -n default get po -l app=frontend -ojsonpath='{.items[0].metadata.name}') \
   -c server -- sh -c 'nc -zv recommendationservice 8080'
   ```

b. Test connectivity across namespaces dev/centos and default/frontend.

   ```bash
   # test connectivity from dev namespace to default namespace, the expected result is "HTTP/1.1 200 OK"
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://frontend.default 2>/dev/null | grep -i http'
   ```

c. Test connectivity from each namespace dev and default to the Internet.

   ```bash
   # test connectivity from dev namespace to the Internet, the expected result is "HTTP/1.1 200 OK"
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://www.google.com 2>/dev/null | grep -i http'
   ```

   ```bash
   # test connectivity from default namespace to the Internet, the expected result is "HTTP/1.1 200 OK"
   kubectl exec -it $(kubectl get po -l app=loadgenerator -ojsonpath='{.items[0].metadata.name}') \
   -c main -- sh -c 'curl -m3 -sI http://www.google.com 2>/dev/null | grep -i http'
   ```

We recommend that you create a global default deny policy after you complete writing policy for the traffic that you want to allow. Use the stage policy feature to get your allowed traffic working as expected, then lock down the cluster to block unwanted traffic.

1. Create a staged global default deny policy. It will shows all the traffic that would be blocked if it were converted into a deny.

   ```yaml
   kubectl apply -f - <<-EOF
   apiVersion: projectcalico.org/v3
   kind: StagedGlobalNetworkPolicy
   metadata:
     name: default-deny
   spec:
     order: 2000
     selector: "projectcalico.org/namespace in {'dev','default'}"
     types:
     - Ingress
     - Egress
   EOF
   ```

   The staged policy does not affect the traffic directly but allows you to view the policy impact if it were to be enforced. You can see the deny traffic in staged policy.

2. Create a network policy to allow the traffic shown as blocked (staged) in step 1, from the centos pod to the nginx in the same namespace.
  
   ```yaml
   kubectl apply -f - <<-EOF   
   apiVersion: projectcalico.org/v3
   kind: NetworkPolicy
   metadata:
     name: default.centos
     namespace: dev
   spec:
     tier: default
     order: 800
     selector: app == "centos"
     egress:
     - action: Allow
       protocol: UDP
       destination:
         selector: k8s-app == "kube-dns"
         namespaceSelector: kubernetes.io/metadata.name == "kube-system" 
         ports:
         - '53'
     - action: Allow
       protocol: TCP
       destination:
         selector: app == "nginx"
     types:
       - Egress
   EOF
   ```

3. Test connectivity with the policy in place.

   a. The only connections between the components within namespaces dev are from centos to nginx, which should be allowed as configured by the policies.

   ```bash
   # test connectivity within dev namespace, the expected result is "HTTP/1.1 200 OK"
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://nginx-svc 2>/dev/null | grep -i http'
   ```
   
   The connections within namespace default should be allowed as usual.
   
   ```bash
   # test connectivity within default namespace in 8080 port
   kubectl exec -it $(kubectl get po -l app=frontend -ojsonpath='{.items[0].metadata.name}') \
   -c server -- sh -c 'nc -zv recommendationservice 8080'
   ``` 

   b. The connections across dev/centos pod and default/frontend pod should be blocked by the application policy.
   
   ```bash   
   # test connectivity from dev namespace to default namespace, the expected result is "command terminated with exit code 1"
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://frontend.default 2>/dev/null | grep -i http'
   ```

   c. Test connectivity from each namespace dev and default to the Internet.
   
   ```bash   
   # test connectivity from dev namespace to the Internet, the expected result is "command terminated with exit code 1"
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -sI http://www.google.com 2>/dev/null | grep -i http'
   ```
   
   ```bash
   # test connectivity from default namespace to the Internet, the expected result is "HTTP/1.1 200 OK"
   kubectl exec -it $(kubectl get po -l app=loadgenerator -ojsonpath='{.items[0].metadata.name}') \
   -c main -- sh -c 'curl -m3 -sI http://www.google.com 2>/dev/null | grep -i http'
   ```


--- 

[:arrow_right: Module 5 - Identity-aware Microsegmentation](/modules/module-5-identity-aware-microsegmentation.md)  <br>

[:arrow_left: Module 3 - Connect the AWS EKS cluster to Calico Cloud](/modules/module-3-connect-calicocloud.md)  
[:leftwards_arrow_with_hook: Back to Main](/README.md)  
