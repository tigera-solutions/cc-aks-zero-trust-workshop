# Module 8 - Clean up

1. Delete the example application stacks.

   ```bash
   kubectl delete -f pre/20-dev-app.yaml
   kubectl delete -f pre/30-onlineboutique-app.yaml
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

4. Delete environment variables backup file.

   ```bash
   rm ~/workshopvars.env
   ```

---

[:leftwards_arrow_with_hook: Back to Main](/README.md)  <br>

[:arrow_left: Module 7 - Zero-trust security controls at application level](/modules/module-7-zero-trust-application.md)  