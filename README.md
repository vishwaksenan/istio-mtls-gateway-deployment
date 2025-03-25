# Istio mTLS Gateway
This repo is a helm chart which will deploy a Istio Secured Gateway enabling mutual TLS.

# How to Deploy 
There are some prerequestics steps to do before installing this helm chart.   
1. Setup your k8s cluster.
2. Install Helm.
3. Install Istio (via Helm).
4. Install Cert manager.
5. Install this helm chart


## Steps
1. Install K8s Cluster (minikube) - [Minikube Installation Guide](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download)
2. Install Helm - [Helm Installation Guide](https://helm.sh/docs/intro/install/).
3. Install Istio - [Istio Installation with Helm Guide](https://istio.io/latest/docs/setup/install/helm/).
4. First install Cert Manager helm chart in your k8s cluster. 
```
helm repo add jetstack https://charts.jetstack.io --force-update
helm install cert-manager jetstack/cert-manager --namespace cert-manager  --create-namespace --version v1.17.0 --set crds.enabled=true
```
5. Create a self signed CA certificate
```
echo "apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-cluster-issuer
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: project1-ca-certificate
  namespace: cert-manager
spec:
  secretName: project1-ca-secret # Secret name
  isCA: true
  commonName: \"project1\"
  duration: 87600h
  renewBefore: 720h
  issuerRef:
    name: selfsigned-cluster-issuer
    kind: ClusterIssuer" | kubectl apply -f -
```
6. Install this helm chart. 
```
helm install istio-mtls . --set=service.port=80
```
7. Get the certificate details
```
kubectl get secret -n istio-ingress project1-ca-secret -o jsonpath='{{.data.tls\.key}}' | base64 --decode > tls.key
kubectl get secret -n istio-ingress project1-ca-secret -o jsonpath='{{.data.tls\.crt}}' | base64 --decode > tls.crt
kubectl get secret -n istio-ingress project1-ca-secret -o jsonpath='{{.data.ca\.crt}}' | base64 --decode > ca.crt
```
8. You can hit the API using
```
curl https://<istio-gateway-ip>/abcdef/ --cacert ca.crt --cert tls.crt --key tls.key
```
