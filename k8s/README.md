# Kubernetes Template for Valheim

## Persistence

The K8S pod expects two PVC's: `valheim-data` and `valheim-config`.  These can be deployed for the first time with `pv.yml`.

The deployment steps are listed in order.  Storage and Secrets are only required prior to the first pod deployment.  Subsequent pod deployments can be performed at any time.

### Storage Deployment

If you're deploying this to a cloud-managed K8S Service, eg AKS, EKS, etc, you'll need to modify `pv.yml` to suit your provider's configuration before deploying.  This `pv.yml` was designed for a bare metal cluster using NFS as the persistent volume.

* `kubectl apply -f pv.yml` to create the PV and PVC required for the pod's persistent storage

### Secrets deployment

You need to deploy the valheim server password as a k8s secret prior to deployment.

`kubectl create secret generic valheim --from-literal=password='PASSWORD_HERE'`

You can retrieve the secret at any time:

`kubectl get secret valheim -o jsonpath='{.data.password}' | base64 --decode`

### NodePort modification

You must modify the NodePort configuration of your `kube-apiserver` prior to deployment, in order for ports to be exposed as expected.

scp `node-port.sh` to the master node(s) and run the script.

### Pod deployment

You'll want to modify the `SERVER_NAME` environment variable value in `deployment.yml` prior to deploying.  It is defaulted to `TEST-K8S-TEST`

You can add additional environment variables as defined by the container's image.

You can perform the initial deployment with: `kubectl apply -f deployment.yml`.

If using MetalLB, forward external direct to 2456,2457,2458 (Or `SERVER_PORT`+0, +1, and +2)

If using cloud-based kubernetes, observe the service ports and forward to those.

### Pod restart

You can restart the pod by running the command below.  

It is possible to perform a `kubectl get pods` and replace `$(kubectl get pod | grep valheim)` with the exact pod name.

`kubectl delete pod $(kubectl get pod | grep valheim)`

### Pod upgrade

In the event the upstream Docker image `lloesche/valheim-server` is updated, you can simply redeploy the pod to refresh Kubernetes' cache:
