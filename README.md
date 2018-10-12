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

Check [this article](https://hackernoon.com/setting-up-nginx-ingress-on-kubernetes-2b733d8d2f45) for how-to. Or it is more convenient to install nginx-ingress by using [helm](https://github.com/helm/charts/tree/master/stable/nginx-ingress).

## How the hell to self-deploy ingress on bare metal without Cloud Provider's support?

Check [this](https://github.com/kubernetes/ingress-nginx/tree/master/deploy)

## How to make ingress-nginx-controller listen to localhost:80

For small clusters which means we don't want to use `LoadBalancer` of the cloud provider, does not even expose port 80 on host network? WTF!

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

Or with helm:

```sh
helm install \
  --set controller.hostNetwork=true \
  --set controller.service.type=NodePort \
  stable/nginx-ingress
```

## Minikube fails to start after (brew cask) upgrade?

```sh
minikube delete
rm -rf ~/.minikube
brew cask reinstall minikube
```

- Maybe you need to terminate the process of VMBox (or other virtual machines) which is locking the old version of minikube
- Maybe you need to upgrade kubernetes

```sh
brew upgrade kubernetes-cli
```

## Helm install: Error: no available release name found

```sh
$ helm install nginx-ingress

Error: no available release name found
```

And also:

## Error: configmaps is forbidden: User "system:serviceaccount:kube-system:default" cannot list configmaps in the namespace "kube-system"

This usually happens when using helm with an old kubernetes version. Solution:

```sh
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```

## What is the install order of helm tiller?

The order of helm install is handled by tiller.

See [here](https://github.com/helm/helm/blob/master/pkg/tiller/kind_sorter.go#L29)

##

# 中国特色

## 因为 helm stable charts registry 被墙，而无法安装 helm charts

```sh
helm init \
  --tiller-image kaelz/tiller:2.11.0 \
  --stable-repo-url https://charts.ost.ai
```

如果之前安装过 tiller，但是 tiller 已经安装失败：

```sh
helm init \
  --tiller-image kaelz/tiller:2.9.1 \
  --upgrade
```

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
