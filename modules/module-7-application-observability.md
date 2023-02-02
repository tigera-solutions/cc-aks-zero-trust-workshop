# Module 7 - Application Level Observability

L7 logs capture application interactions from HTTP header data in requests. Data shows what is actually sent in communications between specific pods, providing more specificity than flow logs. (Flow logs capture data only from connections for workload interactions).

Calico Cloud collects L7 logs by sending the selected traffic through an Envoy proxy.

L7 logs are visible in the Manager UI, service graph, in the HTTP tab.

1. Configure Felix for log data collection

   Enable the Policy Sync API in Felix. For cluster-wide enablement, modify the default FelixConfiguration and set the field policySyncPathPrefix to /var/run/nodeagent.
   
   ```bash
   kubectl patch felixconfiguration default --type='merge' -p '{"spec":{"policySyncPathPrefix":"/var/run/nodeagent"}}'
   ```

2. Configure the ApplicationLayer resource for L7 logs. Ensure that the collectLogs field is set to Enabled.

   ```yaml
   kubectl apply -f - <<-EOF
   apiVersion: operator.tigera.io/v1
   kind: ApplicationLayer
   metadata:
     name: tigera-secure
   spec:
     logCollection:
       collectLogs: Enabled
       logIntervalSeconds: 5
       logRequestsPerInterval: -1
   EOF
   ```
   
   This creates l7-log-collector daemonset in calico-system namespace.
   
   Ensure that the daemonset progresses and l7-collector and envoy-proxy containers inside the daemonset are in a Running state.

3. Select traffic for L7 log collection

   Annotate the frontend service to collect L7 logs as shown.
   
   ```bash
   kubectl annotate svc frontend -n default projectcalico.org/l7-logging=true
   ```

## Service Graph

To view L7 logs in Service Graph:

In the Manager UI left navbar, click Service Graph.

In the bottom pane you will see L7 logs in the HTTP tab.

![http_logs](https://user-images.githubusercontent.com/104035488/216352791-bdbb8376-3b24-4590-81f4-a6b411c1a1cd.gif)

## Kibana dashboards and logs

Kibana is the frontend for Calico Cloud Elasticsearch, which is the logging infrastructure that centrally stores logs from all managed clusters. Kibana provides an interface to explore Elasticsearch logs and gain insights into workload communication traffic volume, performance, and other key aspects of cluster operations. Log data is also summarized in custom dashboards.

### Start Kibana and access dashboards

In Manager UI, from the left navbar select, Kibana. A new browser tab opens into Kibana.

In Kibana, click the hamburger icon in the top left corner, and select Analytics, Dashboard.

![kibana-dashboard](https://user-images.githubusercontent.com/104035488/216352874-83d3dd93-0596-4791-b974-e18c0cd364c7.png)

### L7 HTTP dashboard

The L7 HTTP dashboard provides application performance metrics for inscope Kubernetes services. The data can assist service owners and platform personnel in assessing the health of cluster workloads without the need for a full service mesh. L7 logs are not enabled by default, and must be configured.

![l7-dashboard](https://user-images.githubusercontent.com/104035488/216352987-23be3658-2a66-437f-b791-31340971c287.png)

### DNS dashboard

The DNS dashboard summarizes DNS data and logs into metrics, providing high-level information on the types of DNS lookups made, responses, and overall DNS performance. By default, DNS activity logs are captured only for requests/responses from Kubernetes built-in DNS services (CoreDNS). DNS activity to an external DNS server can be captured by configuring the parameter, dnsTrustedServers in Felix. DNS activity to Node local server is not supported.

![dns-dashboard](https://user-images.githubusercontent.com/104035488/216353120-e78261a0-cd5b-4b89-a171-4225608422c3.png)

---

[:arrow_right: Module 8 - Clean up](/modules/module-8-clean-up.md)    <br>

[:arrow_left: Module 6 - Ingress and Egress access control using NetworkSets](/modules/module-6-network-sets.md)    
[:leftwards_arrow_with_hook: Back to Main](/README.md)  
