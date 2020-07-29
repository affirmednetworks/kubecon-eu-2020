# Setup

## Kubernetes cluster using KinD

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

### Create cluster

```bash
kind create cluster --config ~/.kind/kubecon.config --name kubecon
kubectl cluster-info --context kind-kubecon
```

## Install istio controlplane

```bash
istioctl manifest apply --set profile=demo --set values.gateways.istio-ingressgateway.enabled=false --set values.gateways.istio-egressgateway.enabled=false --set values.global.jwtPolicy=first-party-jwt
```

## Tenants

### tenant-one

```bash
kubectl create ns tenant-one
kubectl label namespace tenant-one istio-injection=enabled
kubectl apply -k tenant-one
```

### tenant-two

```bash
kubectl create ns tenant-two
kubectl label namespace tenant-two istio-injection=enabled
kubectl apply -k tenant-two
```
