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

### Inspect
* See a list of pods: 
`kc get pods`

* Get container logs: 
`kc logs deployment/my-apache --follow --tail 1`, 
`kc logs -l run=my-apache`

* Get a bunch of details about an object, including events: 
`kc describe pod/my-apache-xxxx-zzz`

* Watch a command: 
```kc get pods -w```
But if you delete a pod with `kc delete pod/my-apache-xxxx-zzz`, you can watch it re-create a second pod.

## Services / Exposing Ports
`kc expose` creates a service for existing pods. A **service** is a stable address for pods. If we want to connect to pods, we need a service. CoreDNS allows us to resolve services by name ( a feature of the control plane).

There are different types of services. 
* *ClusterIP* (default): own DNS address, single, internal virtual IP allocated. Only reachable within cluster (nodes and pods). 
* *NodePort*: IP address per node. High port allocated on each node. Port is open on every node's IP. Anyone can connect
* *LoadBalancer*: Mostly used in the cloud. Controls a LB endpoint external to the cluster. Will talk to external provider like AWS ELB etc. Creates NodePort+ClusterIP services, tells LB to send to NodePort. Traffic coming into your cluster from infrastructure.
* *ExternalName*:  Adds CNAME DNS record to CoreDNS only. Not used for pods but for giving pods a DNS name to use for something outside Kubernetes.

### Create ClusterIP Service
Watch the pods with `kc get pods -w`. Then create a deployment to test and create ClusterIP service. 
```
kc create deployment httpenv --image=bretfisher/httpenv
kc scale deployment/httpenv --replicas=5
kc expose deployment/httpenv --port 8888
kc get service
```
Expose command will create a ClusterIP by default in front of the deployment. This is only can be used inside the deployment for nodes or other pods to access. We can reach the IP address only from nodes and pods in the cluster.

We need to run another pod running on the SAME cluster in order to curl the exposed port.
```
kc run --generator run-pod/v1 tmp-shell --rm -it --image bretfisher/netshoot -- bash
```
Output Shell:
```
If you don't see a command prompt, try pressing enter.
bash-5.0# curl httpenv:8888
curl: (6) Could not resolve host: httpenv
bash-5.0# curl 10.152.183.248:8888
{"HOME":"/root","HOSTNAME":"httpenv-6bf64f7c4f-h589k","KUBERNETES_PORT":"tcp://10.152.183.1:443","KUBERNETES_PORT_443_TCP":"tcp://10.152.183.1:443","KUBERNETES_PORT_443_TCP_ADDR":"10.152.183.1","KUBERNETES_PORT_443_TCP_PORT":"443","KUBERNETES_PORT_443_TCP_PROTO":"tcp","KUBERNETES_SERVICE_HOST":"10.152.183.1","KUBERNETES_SERVICE_PORT":"443","KUBERNETES_SERVICE_PORT_HTTPS":"443","PATH":"/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"}bash-5.0# 
```
This error is caused by *microk8s* because by default, the dns service *CoreDNS* is not enabled by default! 
```
microk8s.enable dns
```
### Create a NodePort Service
Creating a NodePort allows the host IP to access containers by a name.
```
kc expose deployment/httpenv --port 8888 --name httpenv-np --type NodePort
```
This new service can be used to access the pod by the port.
```
$ kc get services
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
httpenv      ClusterIP   10.152.183.248   <none>        8888/TCP         26h
httpenv-np   NodePort    10.152.183.201   <none>        8888:30467/TCP   7m56s
kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP          2d2h
```
You can access by port on the right in the `httpenv-np` line. Access by `curl localhost:30467`.

**Kubernetes is customizable by yaml!**

#### Cleanup
```
kc delete service/httpenv service/httpenv-np
kc delete service/httpenv-lb deployment/httpenv
```

### Kubernetes Services DNS
Internal DNS is provided by CoreDNS. Like Swarm, this is DNS-based service discovery. We use hostnames to access services but that only works for services in the same namespace: `kc get namespaces`. Services also have a *Fully Qualified Domain Name*: `curl <hostname>.<namespace>.svc.cluster.local`. (service.<default DNS name>)

## Kubernetes Management
Generators are applied to services, deployments and jobs..
### Generators
These are templates that create "specs" to apply to cluster. Behind commands and provide default configurations. YAML is used for config files and can be seen with the `--dry-run -o yaml` option.
```
kc create deployment sample --image nginx --dry-run -o yaml
```
You can use those YAML defaults as starting points. Generators are "opinionated defaults."
#### Examples
1. See the generators like above. The YAML is *generated* by the command.
```
kc create job test --image nginx --dry-run -o yaml
```
A deployment creates replica sets which create pods. Here a **job** directly creates pods. There is no restart policy with a job.

2. Using expose ( a service )
```
kc create deployment test --image nginx
kc expose deployment/test --port 80 --dry-run -o yaml
```

#### Future of kubectl run
Hopefully generators will be going away. Run should become more like docker run. Run got overly complex, so now create will replace a lot of it and run will only create pods.
```
# today this creates a deployment
kc run test --image nginx --dry-run

# in future, will create pods.

# this will create deployment AND service, both named test.
kc run test --image nginx --port 80 --expose --dry-run

# this will create a batch job! Will on re-create on failure
kc run test --image nginx --restart OnFailure --dry-run

# this will change resource type to pod!
kc run test --image nginx --restart Never --dry-run

# this will change resource type to cronjob!
kc run test --image nginx --schedule "*/1 * * * *" --dry-run
```
### Imperative vs Declarative
* *Imperative* HOW a program operates

Run, create, update are imperative. Easier when you know the state of the resource or when you are just getting started. Easier at the CLI, *NOT* easy to automate.

* *Declarative* WHAT a program should accomplish

We don't know the state, but we do know the end result that we want. This is for automation and the Apply command is the thing: `kc apply -f my-resources.yaml`. Resources can be all in a file and the same command can be repeated. It knows *What* you want and automates it. Requires understanding the YAML keys and values.

**GitOps Happiness**: Edit YAML, Commit, Truth

### Three Management Approaches

* Imperative commands for dev/learning/personal. Hard to manage
* Imperative objects: `create -f file.yml`, `replace -f file.yml`, ..., good for small environments. middle ground that uses yaml/git but hard to automate.
* Declarative objects: `apply -f file.yml`, best for production, easy to automate, hard to understand and predict changes. Same command every time.

**Don't mix three approaches.** But get used to declarative approach for production. Create git repo to store yml files. Learn the imperative CLI for local test setup. Move to `apply -f file.yml` for production and store yaml in git, commit each change before you apply.
