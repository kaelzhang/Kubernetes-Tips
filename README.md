# Kubernetes-Tips
Tips for usage of kubernetes

## The EXTERNAL_IP of LoadBalancer with minikube is always pending

k8s | minikube
---- | ----
`1.9.0` |  `0.25.0`

LoadBalancer services run fine on minikube, just with no real external load balancer created. [via](https://github.com/kubernetes/minikube/issues/384#issuecomment-234409957)

LoadBalancer services get a node port assigned too so you can access services via 

```sh
minikube service <name>
``` 

to open browser or add `--url` flag to output service URL to terminal.
