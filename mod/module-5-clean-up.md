# Module 5 - Clean up

1. Delete the applications stack to clean up any `loadbalancer` services.

   ```bash
   kubectl delete -f pre/004-vote-app-manifest.yaml
   ```

2. Delete the AKS cluster.

   ```bash
   az aks delete \
     --resource-group $RESOURCE_GROUP \
     --name $CLUSTERNAME
   ```

3. Delete the resource group.

   ```bash
   az group delete \
     --name $RESOURCE_GROUP
   ```

5. Delete environment variables backup file.

   ```bash
   rm ~/workshopvars.env
   ```

6. Delete this cloned repository

   ```bash
   cd .. && rm -Rf cc-aks-kspm-compliance-bootcamp
   ```

---

[:arrow_left: Module 4 - Application Level Observability](/mod/module-4-application-observability.md)  <br>

[:leftwards_arrow_with_hook: Back to Main](/README.md)
