# Istio Canary Upgrade

Upgrading using this will be done by first running a canary deployment of the new control plane, allowing you to monitor the effect of the upgrade with a small percentage of the workloads before migrating all of the traffic to the new version. This is much safer than doing an in-place upgrade and is the recommended upgrade method.

## Configure the Helm repository

```bash
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
```
## Installing Canary version

```bash
kubectl apply -f manifests/charts/base/crds
helm install istiod-canary istio/istiod \
    --set revision=canary \
    -n istio-system
```
## Checking Both versions

```bash
kubectl get pods -n istio-system -l app=istiod
kubectl get svc -n istio-system -l app=istiod
kubectl get mutatingwebhookconfigurations
```

## Labelling The NameSpace with Canary Version

```bash
kubectl label namespace test-ns istio-injection- istio.io/rev=canary
kubectl get pods -l app=istiod -L istio.io/rev -n istio-system
kubectl rollout restart deployment -n test-ns

```

## Uninstall the control plane and upgrade the Istio Base Chart

```bash


helm delete istiod -n istio-system
helm upgrade istio-base istio/base --set defaultRevision=canary -n istio-system --skip-crds

```

