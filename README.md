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

The yaml file of `Kind: Ingress` specify the rules of your load balancer.

But the load balancer itself should be provisioned by `ingress-nginx-controller` or other controllers implemented by your Cloud Provider, such as GCE, AWS, Azure, Aliyun, Qcloud, etc.

So, if you want to create an Ingress on your own, you need to deploy an `ingress-nginx-controller` and a service (which is usually listening port `80`) for it.

Check [this article](https://hackernoon.com/setting-up-nginx-ingress-on-kubernetes-2b733d8d2f45) for how-to.

## How the hell to self-deploy ingress on bare metal without Cloud Provider's support?

Check [this](https://github.com/kubernetes/ingress-nginx/tree/master/deploy)

## How to make ingress-nginx-controller listen to port 80

For small clusters, does not even expose port 80 on host network? WTF!

Add `hostNetwork: true` to [`with-rbac.yaml`](https://github.com/kubernetes/ingress-nginx/blob/master/deploy/with-rbac.yaml)

```yaml
...
spec:
  ...
  template:
    ...
    spec:
      serviceAccountName: nginx-ingress-serviceaccount
      hostNetwork: true
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.11.0
...
```

# 中国特色

## 在云服务器上，尝试使用国外镜像，但是由于 “技术原因” 下载失败

常见错误

```
TLS shakehand timeout
```

1. 简单的包装国外镜像

Dockerfile

```Dockerfile
FROM gcr.io/foo/bar:1.0
```

2. 在本地，或者使用能够 “科学上网” 的构建环境构建（docker build）镜像

3. 然后上传到国内的 registry（hub） 或私有 registry。你也可以 [自己搭建](https://github.com/vmware/harbor)。
