# Kubernetes
On ubuntu it is called microk8s. Install by snap and use alias: `alias kc="microk8s.kubectl"`

`kc` will be used in place of `kubectrl` or `microk8s.kubectrl`.

## Pods
The following provides 1 replica of the image.
```
kc run my-nginx --image nginx
kc get pods
```

Pods -> ReplicaSet -> Deployment

A **pod** is a group of containers. Each container runs on docker. Kublets run on each node for managing stuff. Run creates a **deployment** which contains a **ReplicaSet** which contains *pods* which contain *containers*. A replica set permits multiple pods that are both running with identical template. The deployment manages the replica set.

The deployment can be deleted by `kc delete deployment my-nginx`.

Scale it up to 2 like this. This creates 2 *ReplicaSets* and two pods. Updating "deployment spec" by changing replica set to 2 replicas, *ReplicaSet* controller sets pod  count to 2, and the Control plane assigned those two pods to which nodes. Kubelet on the node sees pod is needed and starts container.
```
kc run my-apache --image httpd
kc scale deploy/my-apache --replicas 2
**or** 
kc scale deployment my-apache --replicas 2
```

## Inspect
`kc get pods`