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

## Deploy

```bash
cd deploy/multitenancy
```

### Tenant One

```bash
kubectl create ns tenant-one
kubectl label namespace tenant-one istio-injection=enabled
kubectl apply -k tenant-one
```

```bash
CLUSTER_IP_ONE=$(kubectl -n tenant-one get service istio-ingressgateway -o jsonpath='{.spec.clusterIP}'
```

### Tenant Two

```bash
kubectl create ns tenant-two
kubectl label namespace tenant-two istio-injection=enabled
kubectl apply -k tenant-two
```

```bash
CLUSTER_IP_TWO=$(kubectl -n tenant-two get service istio-ingressgateway -o jsonpath='{.spec.clusterIP}'
```

## Client

```bash
cd deploy
```

```bash
kubectl create ns user
kubectl apply -k sleep
```

## Request

### Tenant One client to server - Successful

```bash
curl -v --resolve "tenant-one.example.com:443:$CLUSTER_IP_ONE" --cacert /etc/certs/cacert --
cert /etc/certs/t1-cert.pem  --key /etc/certs/t1-key.pem  https://tenant-one.example.com:443/productpage
```

### Tenant Two - Successful

```bash
curl -v --resolve "tenant-two.example.com:443:$CLUSTER_IP_TWO" --cacert /etc/certs/cacert --
cert /etc/certs/t2-cert.pem  --key /etc/certs/t2-key.pem  https://tenant-two.example.com:443/productpage
```

### Tenant One client to Tenant Two server - Failure

```bash
curl -v --resolve "tenant-two.example.com:443:$CLUSTER_IP_TWO" --cacert /etc/certs/cacert --
cert /etc/certs/t1-cert.pem  --key /etc/certs/t1-key.pem  https://tenant-two.example.com:443/productpage
```
