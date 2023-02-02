# Module 6 - Ingress and Egress access control using NetworkSets

It's also possible to create network policies for controlling access from and to a specific domain name, or a list of domain names and ip addresses. Let's create some policies to verify this control in action.

1. Implement DNS policy to allow the external endpoint access from a specific workload, e.g. `dev/centos`.

   a. Apply a policy to allow access to `api.twilio.com` endpoint using DNS rule.

   ```yaml
   kubectl apply -f - <<-EOF
   apiVersion: projectcalico.org/v3
   kind: GlobalNetworkPolicy
   metadata:
     name: security.external-domain-access
   spec:
     tier: security
     selector: (app == "centos" && projectcalico.org/namespace == "dev")
     order: 100
     types:
       - Egress
     egress:
     - action: Allow
       source:
         selector: app == 'centos'
       destination:
         domains:
         - '*.twilio.com'
   EOF
   ```
   
   Test the access to the endpoints:

   ```bash
   # test egress access to api.twilio.com
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -skI https://api.twilio.com 2>/dev/null | grep -i http'
   ```

   ```bash
   # test egress access to www.google.com
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -skI https://www.google.com 2>/dev/null | grep -i http'
   ```

   Access to the `api.twilio.com` endpoint should be allowed by the DNS policy and any other external endpoints like `www.google.com` should be denied.

   b. Modify the policy to include `*.google.com` in dns policy and test egress access to www.google.com again.

   ```bash
   # test egress access to www.google.com again and it should be allowed.
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -skI https://www.google.com 2>/dev/null | grep -i http'
   ```

2. Edit the policy to use a `NetworkSet` with DNS domain instead of inline DNS rule.

   a. Apply a policy to allow access to `api.twilio.com` endpoint using DNS policy.

   Deploy the Network Set

   ```yaml
   kubectl apply -f - <<-EOF
   kind: GlobalNetworkSet
   apiVersion: projectcalico.org/v3
   metadata:
     name: allowed-api
     labels: 
       type: allowed-api
   spec:
     allowedEgressDomains:
     - '*.twilio.com'
   EOF
   ```

   b. Deploy the DNS policy using the network set

   ```yaml
   kubectl apply -f - <<-EOF
   apiVersion: projectcalico.org/v3
   kind: GlobalNetworkPolicy
   metadata:
     name: security.external-domain-access
   spec:
     tier: security
     selector: (app == "centos" && projectcalico.org/namespace == "dev")
     order: 100
     types:
       - Egress
     egress:
     - action: Allow
       destination:
         selector: type == "allowed-api"
   EOF
   ```

   c. Test the access to the endpoints.

   ```bash
   # test egress access to api.twilio.com
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -skI https://api.twilio.com 2>/dev/null | grep -i http'
   ```

   ```bash
   # test egress access to www.google.com
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -skI https://www.google.com 2>/dev/null | grep -i http'
   ```

   d. Modify the `NetworkSet` to include `*.google.com` in dns domain and test egress access to www.google.com again.

   ```bash
   # test egress access to www.google.com again and it should be allowed.
   kubectl -n dev exec -t centos -- sh -c 'curl -m3 -skI https://www.google.com 2>/dev/null | grep -i http'
   ```

## Ingress Policies using NetworkSets

The NetworkSet can also be used to block access from a specific ip address or cidr to an endpoint in your cluster. To demonstrate it, we are going to block the access from your workstation to the Online Boutique frontend-external service.

   a. Test the access to the frontend-external service

   ```bash
   curl -sI -m3 $(kubectl get svc frontend-external -ojsonpath='{.status.loadBalancer.ingress[0].ip}') | grep -i http
   ```
   
   b. Identify your workstation ip address and store it in a environment variable

   ```bash
   export MY_IP=$(curl ifconfig.me)
   ```

   c. Create a NetworkSet with your ip address on it.

   ```yaml
   kubectl apply -f - <<-EOF
   kind: GlobalNetworkSet
   apiVersion: projectcalico.org/v3
   metadata:
     name: ip-address-list
     labels: 
       type: blocked-ips
   spec:
     nets:
     - $MY_IP/32
   EOF
   ```
   
   d. Create the policy to deny access to the frontend service.

   ```yaml
   kubectl apply -f - <<-EOF
   apiVersion: projectcalico.org/v3
   kind: GlobalNetworkPolicy
   metadata:
     name: security.blockep-ips
   spec:
     tier: security
     selector: app == "frontend"
     order: 300
     types:
       - Ingress
     ingress:
     - action: Deny
       source:
         selector: type == "blocked-ips"
       destination: {}
     - action: Pass
       source: {}
       destination: {}
   EOF
   ```

   e. Create a global alert for the blocked attempt from the ip-address-list to the frontend.

   ```yaml
   kubectl apply -f - <<-EOF   
   apiVersion: projectcalico.org/v3
   kind: GlobalAlert
   metadata:
     name: blocked-ips
   spec:
     description: "A connection attempt from a blocked ip address just happened."
     summary: "[blocked-ip] ${source_ip} from ${source_name_aggr} networkset attempted to access ${dest_namespace}/${dest_name_aggr}"
     severity: 100
     dataSet: flows
     period: 1m
     lookback: 1m
     query: '(source_name = "ip-address-list")'
     aggregateBy: [dest_namespace, dest_name_aggr, source_name_aggr, source_ip]
     field: num_flows
     metric: sum
     condition: gt
     threshold: 0
   EOF
   ```

   a. Test the access to the frontend-external service. It is blocked now. Wait a few minutes and check the `Activity > Alerts`.

   ```bash
   curl -m3 $(kubectl get svc frontend-external -ojsonpath='{.status.loadBalancer.ingress[0].ip}')
   ```

---

[:arrow_right: Module 7 - Application Level Observability](/modules/module-7-application-observability.md)   <br>

[:arrow_left: Module 5 - Ingress and Egress access control using NetworkSets](/modules/module-5-network-sets.md)   
[:leftwards_arrow_with_hook: Back to Main](/README.md)  