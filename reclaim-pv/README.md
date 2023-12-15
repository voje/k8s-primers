# Reclaim-pv

Scenario:   
* We have a k8s with an existing StorageClass with `Reclaim Policy: Retain`.
* Install an app with persistent storage.
* Data should persist through `helm uninstall <app> && helm install <app>` (`helm uninstall` skips PVC).
* `helm uninsatll` and manually `k delete pvc` (simulate a small misconfiguration disaster).
* Manually recreate PVC.
* Tell helm chart to use the existing PVC.
* Data should persist throughout above scenario.

Commands:
```bash
minikube start

# Storage with Reclaim Policy: Retain
k apply -f local-path-storage.yaml

# Insatll app
helm install pvctest ./pvctest

# Write some data to pod's mounted filesystem
k exec -it pvc-reclaim-test -- /bin/sh -c "touch /var/my-pvc-mnt/$(date -Iseconds)"
k exec -it pvc-reclaim-test -- /bin/sh -c "ls /var/my-pvc-mnt/"

# Delete app
helm delete pvctet

# Make sure PVC persists
k get pvc

# Reinstall app, check data
helm install pvctest ./pvctest
k exec -it pvc-reclaim-test -- /bin/sh -c "ls /var/my-pvc-mnt/"

# Delete app again, delete PVC (oops)
helm delete pvctest
k delete pvc pvc-one

# Recreate PVC using existing PV (<pv-name>)
k get pv
# Set PV name before applying reclaim-pv.yml
k apply -f reclaim-pv.yml

# PVC is created but is "Pending" - we need to change PV's claimRef
k patch pv <pv-name> -p '{"spec":{"claimRef": null}}'

# PVC should be Bound now
k get pvc

# Reinstall app, check data, the createPvc="false" tells 
# the helm chart to use an existing PVC (match PVC name)
helm install pvctest ./pvctest --set createPvc="false"
```
