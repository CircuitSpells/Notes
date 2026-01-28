# Kubernetes

- [example code](https://github.com/nigelpoulton/getting-started-k8s).
- [kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/).
- [kubectl CLI reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands).

## Big Picture

"kubernetes" is often abbreviated to k8s (8 standing for the 8 letters between k and s).

cluster:

- made up of one or more control plane nodes, and a bunch of worker nodes.

control plane nodes:

- schedule tasks, node monitoring, respond to events.

worker nodes:

- user and business apps

H/A means "high availability".

apps are wrapped in containers, containers in pods, pods in deployments. These are all represented in a kubernetes yaml file.

k8s configuration uses a declarative model, but imperative command line syntax is an option.

Pretty much everything in Kubernetes (pods, service object, replica sets, etc.) is a resource in the API.

## Control Plane Nodes

note: used to be called "masters".

it is generally considered good practice not to run user apps on control plane nodes.

per normal H/A standards, 3 control plane nodes is standard. Do not pick an even number; if there is a split in the network, the side that knows it has the majority nodes will arbitrarily pick a new leader. If there are an even number on each side, the nodes will deadlock and go into read-only mode.

on a hosted kubernetes platform, the control plane is hidden from you and is instead managed by the cloud provider.

leader: one control plane node makes active changes to the cluster at any one time. The others are called followers. If the leader goes down, they elect a new one.k

services that make up the control plane:

- kube-apiserver:
  - only access point to the control plane and worker nodes.
  - REST api.
  - consumes json and yaml.
  - when deploying and managing applications, user sends the yaml file describing what's wanted, the api server authenticates, then instructs other control plane features to deploy and manage it.
- cluster store:
  - persists cluster state and config, including the state/config of all apps.
  - based on the etcd NoSql database.
  - performance is critical.
  - you should have backup recovery plans in place (and regularly test them).
- kube-controller-manager
  - controller of controllers: node controller, deployment controller, endpoints controller.
  - watches loops: it watches the parts of the cluster that it is responsible for and looks for changes.
  - reconciles state: ensures that the observed state of the cluster matches the desired state.
- kube-scheduler:
  - watches the api server for work.
  - assigns tasks to worker nodes: affinity/anti-afinity, constraints, taints, resources.

overview to deploy app:

- kubectl command (usually) sends api request.
- request is auth/n/z'd.
- the desired state of the app is written to the cluster store.
- the scheduler farms the work out to worker nodes.
- various controllers sit in watch loops, observing the state of the cluster and making sure it matches what was asked for.

## Worker Nodes

three worker components:

- kubelet:
  - main kubernetes agent (they also runs on cluster nodes).
  - registers node with the cluster (adding CPU, RAM, and other resources to the overall cluster pool so that the scheduler can intelligently assign work to the kubelet or to the node).
  - watches the api server for work.
  - executes pods (recall: a pod is one or more container).
  - reports back to the control plane.
- container runtime:
  - docker used to be baked in, but now the container runtime is pluggable (called the Container Runtime Interface, or CRI).
  - low-level container work.
- kube-proxy:
  - node networking.
  - makes sure every pod running gets an IP address.
  - has its own IP address, and is the one interface to all pods in the worker node.

## Pods

atomic unit of deployment: in VMware is a virtual machine, in Docker it is a container, and in k8s it is a pod.

while k8s does orchestrate containers, those containers must always run inside of pods.

a pod is a shared execution environment (a collection of things needed to run: an IP address in a network port, files from a file system, shared memory).

if two containers are in a single pod, they need their own ports since they share an IP address.

one pod will have one container in a large majority of cases. One example of multiple containers in a pod: a service mesh container that handles networking and telemetry as a gate to the app container.

## Networking

Because pods can spin up/down arbitrarily, their IP addresses are unreliable because they might disappear at any time, or spin up with a new IP entirely. This is solved with a service object which provides a stable IP and a stable DNS name. It also load balances requests to the underlying pods. The service object IP/DNS never changes.

The pods that are assigned to a given service object are determined by labels.
Everything in kubernetes gets labels. The service object looks for pods that share all of the same labels. If a pod has the same labels as the service object, plus a few extra, it will still be included in the service object scope. It a pod contains one but not all labels, it won't be included.

## Deployments

Deployments hold replica sets which hold pods:

- deployment controller:
  - watches api server for new deployments.
  - implements deployments.
  - constantly compares observed state with desired state.
- replica sets:
  - replica count, self-healing, old versions

simple deployment yaml example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
  labels:
    app: web
spec:
  replicas: 5
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      terminationGracePeriodSeconds: 1
      containers:
        - name: hello-pod
          image: path/getting-started-k8s:1.0
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
```

pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod
  labels:
    app: web
spec:
  containers:
    - name: web-ctr
      image: path/getting-started-k8s:1.0
      ports:
        - containerPort: 8080
```

## kubectl CLI

useful commands:

- `kubectl get svc`
- `kubectl get pods --show-labels`
- `kubectl describe svc <service-name>`
- `kubectl delete svc <service-name>`
- `kubectl get ep` (get endpoints)
- `kubectl explain <name-of-field>`
  - e.g. `kubectl explain Pod.spec.containers`

## Creating Services

### Create a Service Imperatively

_imperative_ way of creating a public port to access a node. Recall that nodes are not accessed directly; a service is set up that acts as an access point to its nodes:

```
kubectl expose pod hello-pod \
--name=hello-svc \
--target-port=8080 \
--type=NodePort
```

the CLUSTER-IP is the external access point. You will have to append the node port, e.g. `85.159.209.35:31779`, to access the web server.

node ports are automatically assigned between 30000 and 32767.

delete the service:

```
kubectl delete svc hello-svc
```

### Create a Service Declaratively

config file for service:

```yml
apiVersion: v1
kind: Service
metadata:
  name: ps-nodeport
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 31111
      protocol: TCP
  selector:
    app: web
```

a note on `spec.type` above: there are three options:

1. `ClusterIP` (default): gives a stable IP within the cluster (service is only available within the cluster).
2. `NodePort`: builds on ClusterIP and adds a port number, allowing for access from outside the cluster.
3. `LoadBalancer`: builds on ClusterIP and NodePort, allowing for external access via your cloud provider's load balancer.

Request flow: there are multiple worker nodes listening publicly on port `31111`. An external service hits that port. One of the worker nodes forwards this request to the service, which is sitting at the autogenerated ClusterIP at port `80` inside the cluster. Recall that the service oversees pods. One of the pods has a container app listening to port `8080`. The service forwards the request to that app.

`spec.selector.app` is a list of labels that has to match the labels on the pod we deployed earlier. You can check labels on a pod with the following command:

```
kubectl get pods --show-labels
```

to deploy the svc-nodeport.yml file:

```
kubectl apply -f svc-nodeport.yml
```

once deployed, check the service (note `Endpoints`: this shows a list of healthy pod IPs that match the label selector):

```
kubectl describe svc ps-nodeport
```

you can also check the endpoints directly with the following:

```
kubectl get ep
```

### Create a Cloud Load Balancer Service

config file for load balancer service:

```yml
apiVersion: v1
kind: Service
metadata:
  name: ps-lb
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app: web
```

deploy:

```
kubectl apply -f svc-lb.yml
```

check:

```
kubectl get svc
```

note that the load balancer automatically assigns a public IP.

## Creating Deployments

deployments are a k8s resource.

deployments handle rollouts and rollbacks. Replica sets handle reliability and scaling.

typical flow: you deploy a deployment yaml file. It talks to the cluster API entry point, is authenticated/etc, at which point the new replica set is created in the cluster. This replica set, say, spins up 5 new pods. Then, when a new version is ready, the deployment yaml file is updated, and deployed again. This creates a new replica set, which begins to start swapping out the old pods with the new version defined in the new replica set. Something important: the old replica set remains. This is so that if there is a need to roll back, the process is very smooth. This whole process can be customized, e.g. waiting 10 minutes before updating each pod on a new deployment to ensure stability.

config file for deployment:

```yaml
# prettier-ignore
apiVersion: apps/v1                    # deployment spec
kind: Deployment                       # |
metadata:                              # |
  name: web-deploy                     # |
  labels:                              # |
    app: web                           # |
spec:                                  # |
  replicas: 5                          # |
  selector:                            # |
    matchLabels:                       # |
      app: web                         # |
  template:                              # pod spec
    metadata:                            # |
      labels:                            # |
        app: web                         # |
    spec:                                # |
      terminationGracePeriodSeconds: 1   # |
      containers:                          # container spec
        - name: hello-pod                  # |
          image: nigelpoulton/getting-started-k8s:1.0
          imagePullPolicy: Always          # |
          ports:                           # |
            - containerPort: 8080          # |
```

note that the deployment spec contains the deployment spec, the pod spec, and the container spec. The pod spec contains the pod spec and the container spec. The container spec contains the container spec.

- `spec.replicas` defines the number of identical pods/containers defined at `spec.template.spec.containers`.
- `spec.selector.matchLabels` is how the deployment knows which pods to work on during things like rolling updates. It has to match the labels in `spec.template.metadata.labels`. Note that the top label at `metadata.labels` is overall less important, and is used for CLI tools for grouping, etc.
- `spec.template.spec.containers.imagePullPolicy` set to `Always` means it will always pull from the registry instead of locally, which protects against malicious attacks where someone puts an image with the same name locally.

### Deploying Deployments

say we created the load balancer service from earlier:

```
kubectl apply -f svc-lb.yml
```

now we can apply the deployment:

```
kubectl apply -f deploy.yml
```

this will have created 5 pods, all running the same image:

```
kubectl get pods
```

to inspect the deployment and replica sets:

```
kubectl get deploy
kubectl get rs
```

the replica set name will be the name of the deployment followed by a hash of the pod spec.

if we now look at the load balancer service:

```
kubectl describe svc ps-lb
```

we can see that the `Selector` field shows the correct label. Verify the label on the pods:

```
kubectl get pods --show-labels
```

see the endpoints directly here:

```
kubectl describe ep ps-ls
```

### Self-Healing

use `kubectl get pods` and copy a pod name, then run:

```
kubectl delete pod <pod-name>
```

if you run `kubectl get pods` soon thereafter, you will notice there are still 5 pods due to the replica set self-healing. The same will happen if nodes are deleted; the pods the node holds will be deleted, but the replica set will work with the cloud provider to provision the number of specified nodes, then will spin up the specified number of pods in `deploy.yml`.

### Scaling

say that `deploy.yml` is updated so that `spec.replicas` is now 10.

running `kubectl apply -f deploy.yml` will spin up 5 new pods. Similar behavior for scaling down.

### Rollouts

say the deploy config was updated to add `spec.minReadySeconds` and `spec.strategy`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
  labels:
    app: web
spec:
  selector:
    matchLabels:
      app: web
  replicas: 10
  minReadySeconds: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  template:
    metadata:
      labels:
        app: web
    spec:
      terminationGracePeriodSeconds: 1
      containers:
        - name: hello-pod
          image: nigelpoulton/getting-started-k8s:2.0
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
```

- `spec.strategy.type` set to `RollingUpdate` will update the pods one at a time any time the pod spec is updated, as opposed to all at once.
  - `maxUnavailable` says how many pods below `spec.replicas` can we go. In this case, we cannot go below 10 pods.
  - `maxSurge` says how many pods above `spec.replicas` can we go. In this case, we can go up to 11 pods.
- `spec.minReadySeconds` is how long a new pod needs to be running before the next pod is terminated/spun up.

run the deployment:

```
kubectl apply -f deploy.yml
```

to watch pods as they're added/removed:

```
kubectl get pods --watch
```

in a separate terminal window, see how many of the pod deployments have completed:

```
kubectl rollout status deploy web-deploy
```

note: if you begin to try and access the pods (say they're hosting a frontend or something) during the deployment, you will randomly begin to see both the old and new version.

### Rollbacks

recall that old replica sets stick around (you can see which one is newer with the `AGE` field):

```
kubectl get rs
```

see rollout history and revision numbers:

```
kubectl rollout history deploy web-deploy
```

to rollback:

```
kubectl rollout undo deploy web-deploy --to-revision <number>
```

a rollback is simply a rollout in reverse.
