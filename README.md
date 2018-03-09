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

## How to use local images for a Deployment

Specify `imagePullPolicy: IfNotPresent` for your container

```yaml
kind: Deployment
...
spec:
  ...
  template:
    spec:
      containers:
        - name: NAME
          image: IMAGE_NAME
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
```

## Why there is no `ADDRESS` of my Ingress when kubectl get ing?

The yaml file of `Kind: Ingress` specify the rules of your load balancer. But the load balancer itself is provisioned by `ingress-nginx-controller` or other controllers implemented by your cloud service, such as GCE, AWS, Azure, Aliyun, Qcloud, etc.

So, if you want to create an Ingress on your own, you need to deploy an `ingress-nginx-controller` and a service (which is usually listening port `80`) for it.

Check [this article](https://hackernoon.com/setting-up-nginx-ingress-on-kubernetes-2b733d8d2f45) for how-to.
