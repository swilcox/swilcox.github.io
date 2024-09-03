+++
title = 'Hello World in k3s (Slightly Updated)'
date = 2024-09-02T18:51:59-05:00
draft = false
tags = ['k3s', 'k8s', 'kubectl', 'nginx', 'traefik']
summary = 'A slightly updated hello, world test for k3s'
+++
# Hello World in K3S (slightly updated)

So messing with [k3s](https://docs.k3s.io/) on one of my home linux boxes and googled for a hello-world type of example just to make sure it was running (I'd just uninstalled and re-installed everything). I was somewhat surprised to see the top result was [Jeff Geerling's post about it](https://www.jeffgeerling.com/blog/2022/quick-hello-world-http-deployment-testing-k3s-and-traefik). I'm a big fan of Jeff's as I've been watching [his YouTube videos](https://www.youtube.com/@JeffGeerling) for quite some time. Anyway, after looking over his post, I decided to update his scripts slightly with a minor fix, some references to namespacing and also deleting it, just for completeness sake.

## Initial Setup and Overview

We're assuming you already have [k3s](https://docs.k3s.io/quick-start) (or something k8s flavor installed and configured enough that you can do `kubectl` commands).

What are we creating? In short -- we're deploying nginx pods (3 replicas) and handling incoming traffic on port 80 (non-TLS) to route to those pods. To try to use some official kubernetes terms. We'll be defining:
* [service](https://kubernetes.io/docs/concepts/services-networking/service/) (called `hello-world`)
* a [deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) (called `hello-world-nginx`) :arrow_right: 
* a `hello-world` [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) which is defined as just the nginx container. We're only exposing port 80 in the container. 
* a [volume mount](https://www.kubermatic.com/blog/keeping-the-state-of-apps-1-introduction-to-volume-and-volumemounts/) where the `hello-world-volume` is mounted at `/usr/share/nginx/html` (which is the default path where nginx looks for static content to serve). The volume defined using a [configMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) which is also being called `hello-world`. 

{{< callout note note >}}
We'll just use the crap out of the name, `hello-world`, because *naming things is hard*:tm:. So almost everything is named `hello-world`. It might have been interesting to name things based on the type of resource they were, e.g. `hello-world-replicaset` or `hello-world-volume` but since that's maybe not so typical, just kept it mostly as Jeff had it which tends to line up with other examples in the documentation as well. But it's good to keep in mind the different *things* we're defining here.
{{< /callout >}}

{{< callout important recommendation >}}
I recommend putting the files you're going to create in their own directory for your own sanity.
{{< /callout >}}

## The Namespace

So the first thing I would tend to do is to create [namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) for this service and deployment to run in so that's not in just the default or current namespace.

```shell
kubectl create namespace hello-world
```

Once again, we're abusing the `hello-world` name and also using that as our namespace to run in.

At this point, you *could* then also set that to be the default namespace in your current context, but I'm skipping that, and just specifying the `--namespace` on each of these commands. If you want to just use the `default` namespace, you can leave off the `--namespace` part.

## The Content to Serve / ConfigMap

This is for the volume mapping / mount, etc. It also happens to be our actual content.

Save this file as `hello-world.html`

```html
<html>
<head>
  <title>Hello World!</title>
</head>
<body>Hello World!</body>
</html>
```

Then to create a ConfigMap based off that file:

```shell
kubectl create configmap hello-world --from-file index.html --namespace=hello-world
```

## Service / deployment

Create a file `hello-world.yml`:

```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-world
  annotations:
    spec.ingressClassName: "traefik"
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: hello-world
            port:
              number: 80

---
apiVersion: v1
kind: Service
metadata:
  name: hello-world
spec:
  ports:
    - port: 80
      protocol: TCP
  selector:
    app:  hello-world

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-nginx
spec:
  selector:
    matchLabels:
      app: hello-world
  replicas: 3
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: hello-world-volume
          mountPath: /usr/share/nginx/html
      volumes:
      - name: hello-world-volume
        configMap:
          name: hello-world
```

Note, my fix here from Jeff's version is line 7:
`spec.ingressClassName: "traefik"`

To now deploy this:

```shell
kubectl apply -f hello-world.yml --namespace=hello-world
```

Once that executes correctly, from *any* machine on the network that can reach your server running k3s, you should be able to hit port 80 and get the hello world web page above:

```shell
curl yourserver.local:80
```

## Deleting

Essentially we can just go in reverse:

```shell
kubectl delete -f hello-world.yml --namespace=hello-world
```

That should stop and remove the pods and service.

```shell
kubectl delete configmap hello-world --namespace=hello-world
```

That should delete the configmap we created.

```shell
kubectl delete namespace hello-world
```

This :arrow_up: will delete the namespace if you want to keep your namespaces clean.

## Other Things

It would be nice to develop an example with:
* TLS
* [metallb](https://metallb.universe.tf/)

