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

## Declarative Kubernetes
```
kc apply
k8s config yaml
build yaml
build yaml spec
dry runs and diffs
labels and annotations
```

### Apply
```
kubectl apply -f filename.yml
kubectl apply -f folder/
kubectl apply -f https://bret.run/pod.yml ## similar to curl -L https://bret.run/pod.yml

```
One command for everything. Replaces `kc create`, `kc replace` and `kc edit`.
### Configuration of YAML
```
$ curl -L https://bret.run/pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.17.3
    ports:
    - containerPort: 80
```
YAML is easier for humans to read than JSON. It will be converted by Kubernetes. Each file contains one or more **Manifests**. Each manifest needs four parts (root key:values in the file): *apiVersion*, *kind*, *metadata*, and *spec*. (also see [resources/k8s-yaml/](./resources/k8s-yaml/))

### Building YAML Files
* **kind**: we can get a list of resources the cluster supports. `kc api-resources` shows the list of sub-options for the kind field. Notice some resources have multiple API's (new vs old)
* **apiVersion**: we can get the API versions the cluster supports. `kc api-versions` shows the different versions of APIs we need to worry about. Don't use the beta versions. YAML will break in several releases.. These are values for the *apiVersion* field in YAML. The `Kind + ApiVersion` determines resource and version for that resource.
* **metadata**: only name is required
* **spec**: where all the actions are, including containers to be created. See next section

### Building YAML Spec
We can get all the keys each **kind** supports by ` kc explain services --recursive`. This will list services available. Find more information about what it does by `kc explain services.spec`. Provides descriptions of each portion under spec. You can drill down further by `kc explain services.spec.type`. These are the man pages for Kubernetes. You can look at further subcategories as noted. *i.e.* `kc explain deployment.spec.template.spec.volumes.nfs.server`. We can also use docs online at [kubernetes.io/docs/reference/#api-reference](kubernetes.io/docs/reference/#api-reference).

### Dry Runs and Diffs
Dry run will go to server and see what needs to happen and report back the diff. In [resources/app.yml](./resources/k8s-yaml/app.yml) uses **selector** to find a resource. Compare
```
kc apply -f app.yml --dry-run
kc apply -f app.yml --server-dry-run

# see the diff
kc diff -f app.yml
```
The diff looks at the YAML on record and the YAML on your local machine and provides a literal diff.
### Labels and Label Selectors
**Labels** go under *metadata* in your YAML. It is a list of `key: value`  for identifying your resource later by selecting, grouping, or filtering it. Examples include: `tier: frontend`, `app: api`, `env: prod`, `customer: acme.co`. Not meant for large, complex, non-identification information. You can filter pods to get a command: `kc get pods -l app=nginx`.

**Label Selectors** are the glue telling services and deployments which pods are theirs. *Which pods belong to me?* Many resources use these to "link" resource dependencies. The term *selector* is used in the YAML file to direct traffic. The spec will match the label.. between Service and Deployment, see [resources/k8s-yaml/app.yml](./resources/k8s-yaml/app.yml) for example. They can be used to control which pods go to which nodes.

## Future of Kubernetes

* *Storage in Kubernetes*: containers are stateless. K8s uses a new resource type called StatefulSets that makes Pods more "sticky." Not recommended for beginners -- use DaaS like cloud databases instead. You can add volumes in K8s. There are two types of volumes:
  * *Volumes* are tied to a Pod. All containers in a Pod can share them.
  * *PersistentVolumes* are defined at the cluster level. Multiple pods can share them, outlives a Pod. Separates storage config from the Pod using it. The Pod makes a claim and storage is provided.
  * Also see CSI plugins as a new way to connect to storage.
* *Ingress*: None of our Service types work at OSI Layer 7 (HTTP). We need to route outside connections based on hosname or URL. Ingress Controllers do this with 3rd party proxies. Also see Nginx, Traefix, HAProxy, F5, Envoy, Istio, etc..
* *CRD's and the Operator Pattern*: You can add 3rd party Resources and Controllers, this extends the K8s API and CLI. *Operator*: automate deployment and management of complex apps. Allows operation of other opensource stuff from K8s such as databases, monitoring tools, backups, custom ingresses, etc.
* *Higher Deployment Abstractions*: all our *kubectl* commands talk to K8s API. There are ways to add applications on top of K8s in order to deploy such as **Helm** and *JenkinsX*. Helm allows you to create templates to simplify deployment. Creates YAML for you. K8s preference is that you use **Compose on Kubernetes** which comes with Docker Desktop. You can choose your orchestrator. **Translates Compose YAML to K8s YAML**
  * All these tools are about *templating YAML*. **Helm** is the best solution today. Look into the `docker app` subcommand.
* *Kubernetes Dashboard*: a gui, yuck.
* *Kubectl Namespaces and Context*: namespaces limit scope, aka "virtual clusters". Some are built in and you can see `kubectl get namespaces`. Context changes `kubectl` cluster and namespaces. See `~/.kube/config` file or `kubectl config get-contexts`.
* *K8s* can be the "differencing and scheduling engine backbone" for many different projects. See knative, k3s, k3OS, service mesh