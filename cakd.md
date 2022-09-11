# Kubernetes Objects

Kubernetes objects are created or managed through an input file
which is provided in yaml.

It is yaml.

```yaml
appVersion:
kind:
metadata:

spec:
```

or

```yaml
appVersion:
kind:
metadata:

data:
```

All kubernetes objects have the same configuration as above.
The `appVersion` value depends on the kind of the object created. The following are `appVersion` according to its `kind`.

| kind       | appVersion |
| ---------- | :--------: |
| Pod        |     v1     |
| Deployment |  apps/v1   |
| ReplicaSet |  apps/v1   |
| Service    |     v1     |

While the first three top fields are same for all
of the kubernetes objects, the fourth fields can be different.
Especially when it is `Secrets` or `ConfigMaps`, this field will be `data`

# POD

In kubernetes the container is encapsulated into a `Pod`. A pod is a single instance of application. A pod is a smallest object created in the kubernetes.

```yaml
appVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: myapp
    tier: frontend
spec:
  securityContext:
    runAsUser: 1000
  containers:
    - name: app-container
      image: app
      command: ["sleep", "20"]
      securityContext:
        runAsUser: 1010
        capabilities:
          add: [MAC_ADMIN, SYS_TIME]
  serviceAccount: dashboard-sa
```

## Command to Interact with Pod

### Create pod

```yaml
kubectl create -f <pod-definition>.yaml
```

The imperative command to do this is,

```yaml
kubectl run --image=<image_name> --generator=run-pod/v1 <pod-name>
```

### Edit pod

In a situation where the currently running pod needs to be updated and assuming no pod definition file is given.

> Assuming we want to update the modifiable attributes.
> What we can to is to edit the pod directly

```yaml
kubectl edit pod <pod-name>
```

> Assuming we want to update the non-modifiable attributes, such as `command`, `serviceAccount`, `securityContext` or even `image`.
> What we can do is

- generate the pod definition yaml file from existing pod

```yaml
kubectl get pod -o yaml > pod_def_file.yaml
```

- delete the existing pod

```yaml
kubectl delete pod <pod_name>
```

- create the new pod

```yaml
kubectl create -f pod_def_file.yaml
```

# ReplicaSet

# Deployment

# ConfigMaps

## Create ConfigMap

- Declarative Way

  > Define a ConfigMap definition file. Where it has `apiVersion`, `kind`, `metadata`, `data` (instead of `spec`)

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: fruit-config
  data:
    sweet: grape
    sour: soursop
  ```

  The command to create it in declarative way is,

  ```bash
  kubectl create -f fruit.yaml
  ```

- Imperative

  ```bash
  kubectl create configmap \
  <config-name> --from-literal=<key>=<value>
  ```

  `Example`

  ```bash
  kubectl create configmap \
  fruit-config --from-literal=sweet=grape \
               --from-literal=sour=soursop
  ```

  > this will be more complicated when there is a lot of configuration item, 
  so that there is other option supported `--from-file` which accepts the file path of a configuration file

  ```bash
  kubectl create configmap \
  fruit-config --from-file=fruit.properties
  ```
> Another option is when you have several files in a directory, the config map can be created by feeding the command with
the directory path.

```
config
  engine.conf
  api.conf
```

The config map can be created by,

```
kubectl create configmap app-config --from-file=config
```




## Inject into the POD

# Secrets

# ServiceAccount
We've been using the `kubectl` executable to run operations againts a Kubernetes cluster. Under the hood, its implementation calls the API server by making an HTTP call to the exposed endpoints. Some applications running inside of a Pod may have to communicate with the API server as well.
For example, the application may ask for specific cluster node information or available namespaces.

Pods use a Service Account to authenticate with the API server through an authentication token. A kubernetes administrator assigns rules to a Service Account via role-basedd access control (RBAC) to authorize access to specific resources and actions.

# SecurityContext

# Resource Limit


`Resource Boundaries` 
Namespaces do not enforce any quotas for computing resources like CPU, memmory or disk space, nor do they limit the number of Kubernetes object that can be created. As a result, Kubernetes objects can consume unlimited resources until the maximum available capacity is reached.
 
`ResourceQuota`

Kubernetes primitive that establishes the usable, maximum amount of resources per namespace.
Once put in place, the kubernetes scheduler will take care to enforce those rules.

>Kubernetes measures CPU resources in millicores and memory resources in bytes. Eg, `600m` or `100Mib`

Typical rule enforced by the `ResourceQuoate`

* Setting an upper limit for the number of objects that can be created for a specific type (e.g. , a maximum of 3 Pods)

* Limiting the total sum of compute resources (e.g. 3 GiB of RAM)
* Expecting a Quality of Service (QoS) class for Pod (e.g., BestEffort to indicate that the Pod must not make any memory or CPU memory limits or requests)

For example, below is the resource limit requirement manifest given for a ckad namespace

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ckad-quota
  namespace: ckad
spec:
  hard:
    pods: 2
    requests.cpu: "1"
    requests.memory: 1024m
    limits.cpu: "4"
    limits.memory: 4096m
```












# Taint and Toleration

Toleration is the restriction on what `pod` can be secheduled on a `node`. It is nothing to do with security. It is not about which node a pod should go, but is it about if a particular `node` accepts a `pod`. It means that in case of a `pod` is tolerant to a `node` it is accepted by that particular `node`,however it is not guaranteed to be scheduled on the node. The other node with no **taint** can accept the same pod.
`Taints` are set on `node` while `tolerations` are set on `pod`
Few use cases apply,

- A certain node should be scheduled for any `pod`, eg. `master node`

  > The master node should be `taint`ed so, and no pods are `tolerant`

- Some nodes is dedicated to schedule a certain `pod` and the rest are for others.
  > Set a certain node to be `taint`ed. Only `pod` with tolerant level will be scheduled in this node

## How to Use ?

There are two ways to apply the toleration on a pod

### Enable the taint on the node

```yaml
kubectl taint nodes <node-name> key=value:taint-effect
```

Example

```yaml
kubectl taint nodes node01 spray=mortein:NoSchedule
---
node/node01 tainted
```

Which means to set the `taint` **blue** on `node` **node1**, and no `pod` should be scheduled on **node1** if it is not `tolerant`.

### Enable the toleration on the pod

For a `pod` meets the toleration level set on the `node`, the pod definition should include the toleration with the `taint` value
match with the one set on the node.

**Example**

```yaml
appVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: myapp
    tier: frontend
spec:
  securityContext:
    runAsUser: 1000
  containers:
    - name: app-container
      image: app
      command: ["sleep", "20"]
      securityContext:
        runAsUser: 1010
        capabilities:
          add: [MAC_ADMIN, SYS_TIME]
  serviceAccount: dashboard-sa
  tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
```

where `taint-effect` defines _what happen to the `pods`_ that do not `tolerate` this `taint`.

- NoSchedule
  > The pod will not be scheduled
  > Error Message on the pod creation will be something like,

```yaml
  Warning  FailedScheduling  <unknown>  default-scheduler  0/2 nodes are available: 2 node(s) had taints that the pod didn't tolerate.

  Warning  FailedScheduling  <unknown>  default-scheduler  0/2 nodes are available: 2 node(s) had taints that the pod didn't tolerate.
```

- PreferNoSchedule
  > The controller will try to not schedule but it is not guaranteed
- NoExecute
  > The new pod will not be scheduled and the existing pod will be deleted if it is not tolerant. As some pods may have been scheduled before the taint is applied

### Master Node

Master nodes has taint with the taint effect `NoSchedule` so that
no pod will be scheduled on this node. Run this command to find out how the taint is set on master node

```yaml
kubectl describe nodes kubemaster |grep Taints

Taints:             node-role.kubernetes.io/master:NoSchedule
```

### Removing the Taints

```yaml
master $ kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-

node/master untainted
```

### To check which node a pod is running

```yaml
master $ kubectl get pods -o wide
NAME       READY   STATUS    RESTARTS   AGE   IP          NODE     NOMINATED NODE   READINESS GATES
bee        1/1     Running   0          11m   10.44.0.1   node01   <none>           <none>
mosquito   1/1     Running   0          20m   10.32.0.3   master   <none>           <none>
```

# Pod Scheduling Management

## Node Selectors

---

The `nodeSelector` achieves the simple requirement for choosing which node should a pod be scheduled. How it works is explained in the following.

Label the node so that the pod with `nodeSelector` definition will be scheduled in that particular node. For example, in a deployment, we have 3 nodes with different computing power capacity. The application that requires more resoueces should be deployed into the node with more capacity. The simplest way to achieve this is by using `nodeSelector`

### Label the Node

```yaml
kubectl  label node <node-name> <label-key>=<label-value>
```

Example

```yaml
kubectl  label node node01 size=Large
node/node01 labeled
```

### Define Pod with node Selector

```yaml
appVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: myapp
    tier: frontend
spec:
  securityContext:
    runAsUser: 1000
  containers:
    - name: app-container
      image: app
      command: ["sleep", "20"]
      securityContext:
        runAsUser: 1010
        capabilities:
          add: [MAC_ADMIN, SYS_TIME]
  serviceAccount: dashboard-sa
  nodeSelector:
    size: Large
```

The pod with `nodeSelector` _size=Large_ is now scheduled in the node with the same label. As the requiment for getting a particular application only deployed into a dedicated node, this requirement will be getting more complex. For example, what if the pod in the previous example should be deployed into a non "Small" size of node? The `nodeSelector` is not for this and need other solution.

## Node Affinity

---

> With great power comes great complexity

The `Node Affinity` comes with the complex expression to decide on which node a pod should be scheduled. Assuming a node has labels accordingly.
The last requirement in the `Node Selector` section, cannot achieve the following requirement,

> If a pod should be scheduled on a node which its label is either `size=Medium` or `size=Large`

`nodeAffinity` comes with this rule given as the affinity expression.

```yaml
appVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: myapp
    tier: frontend
spec:
  containers:
    - name: app-container
      image: app
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: In
                values:
                  - Large
```

In case the requirement says that the pod only can be deployed into node which is not small then the affinity rule expression should something like

```yaml
appVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: myapp
    tier: frontend
spec:
  containers:
    - name: app-container
      image: app
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: NotIn
                values:
                  - Small
```

Assuming that we only label the `Large` and `Medium` nodes, the `small` one is without label. The same requirement is achieved by this affinity expression,

```yaml
appVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: myapp
    tier: frontend
spec:
  containers:
    - name: app-container
      image: app
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: Exists
```

The pod will be scheduled on node with label `size=Large` or `size=Medium`.

### Node Affinity Type

---

It is about how affinity rules affect the pod on its lifecycle.
Concerning the affinity rules, there are two stages on pod lifecycle

| DuringScheduling | DuringExecution |  Type  |
| :--------------- | :-------------: | :----: |
| Required         |     Ignored     | Type 1 |
| Preferred        |     Ignored     | Type 2 |
| Required         |    Required     | Type 3 |

```yaml
appVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: myapp
    tier: frontend
spec:
  containers:
    - name: app-container
      image: app
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution: #this is affinity type
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: Exists
```

> DuringScheduling
> How the affinity affects a pod during scheduling where the pod does not exist in the beginning.
> For the `RequiredDuringScheduling`, if no node with corresponding label exists, the pod will not be scheduled
> For the `PreferredDuringScheduling`, the controller will try its best to find the node with correct label, if it is not found, the pod will be placed to any nod

> DuringExecuting
> How the affinity affects a pod during executing while the pod is running, the node's label is getting changed.
> Right now, all available types are with `IgnoredDuringExecution`, which means the pod will continue running. The planned feature is to have `RequiredDuringExecution`

# Node Affinity - Taint/Toleration

Use case

> Assuming we shared the kubernetes cluster with different team.
> We have several dedicated nodes for our team, and we do not want any pods other than our pods deployed into those nodes. On top of this, each of our pod has its own dedicated node

Why `affinity` alone will not solve the problem ?

> While our pods with affinity expression will be well placed on each of dedicated nodes, it will not guarantee the other pod without `affinity` will not be placed in those nodes. The missing part is how to prevent pod without afinity? So it is about restricting a node from being accepting a particular pod.

Why `taint - toleration` alone will not solve the problem either ?

> While each node with corresponding taint will not accept the the pod with incorrect tolerant, however, the tolerant pod is not guaranteed to be scheduled on the node with taint. It will not prevent the pod from being scheduled into the other pods with no taint. The missing part is how to guarantee the tolerant pod to only go to the specifif pod, and not to other pod. So it is about restricting a pod so that it can only go to a particular node.

The combination of `affinity` and `taint-toleration` is the answer.

- Set node with taint
- Set node with label
- Set pod with toleration
- Set pod with affinity

# Multi Container Pod

It comes with pattern

## SideCar

---

Assuming the web server container with log agent container in a single pod

## Ambassador

---

Assuming the db connectivity is established to a different container in a a pod

## Adapter

---

In case of log agent, where agent in each pod send the log to central logging. The format need to be the same. To adapt that, an adapter necessary in each pod. In case this adapter is a separate container, then this is adapter design pattern

# Liveness and Probe

## Pod Status

At any point of time, a pod can only be in the following status.

- Pending

> When the scheduler is in process to decide in which node the pod will be scheduled.

- ContainerCreating

> When a pod is scheduled in a node and waiting until the container starts

- Running

> When the container in a pod is started. This status will remain until the program in the pod completed or a pod is terminated

- Terminated

## Pod Condition

The condition complements pod status

- PodScheduled
  `true` if the pod is scheduled
- Initialized
  `true` if the pod is initialized
- ContainersReady
  `true` if all containers in a pod is created
  This is applicable for a `multi container` pod
- Ready
  `true` if the pod is ready
  This is applicable for a `single container` pod

The problem is that the application inside the container takes more time to get ready. Just because the container is created, it does not mean that the application is ready to receive the request.
To tie the pod readiness state and the application readiness is an application specific.

## Few Ways To Implement Readiness Probe

It depends on the application,

- HTTP Test

> GET /api/ready
> In pod the `readiness probe` is part of the containers block.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
      readinessProbe:
        httpGet:
          path: /api/ready
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 5
        failureThreshold: 8
```

Upon the container created, the kubernetes will not set the ready status to `ready`. But it will perform the test by calling the ready API. Until it gives the success response then only the ready status is set to `ready`.

- TCP Test

> check if a particular TCP socket in a given port is listening

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-socket
  labels:
    name: simple-socket
spec:
  containers:
    - name: simple-socket
      image: simple-socket
      ports:
        - containerPort: 3306
      readinessProbe:
        tcpSocket:
          port: 3306
```

- Exec Command

> Execute the command on the container

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-shell
  labels:
    name: simple-shell
spec:
  containers:
    - name: simple-shell
      image: simple-shell
      readinessProbe:
        exec:
          command:
            - cat
            - /app/is_ready
```

# Deployment Strategy

## Recreate

## Rolling Upgrade (default)
