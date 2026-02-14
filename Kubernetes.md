# Kubernetes

- [k8s installation guide](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/).
- [example code](https://github.com/nigelpoulton/getting-started-k8s).
- [more example code](https://github.com/nigelpoulton/ps-vols-and-pods).
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

pretty much everything in Kubernetes (pods, service object, replica sets, etc.) is a resource in the API.

use the Lens application for a Kubernetes GUI.

## kubectl CLI

useful commands:

- `kubectl cluster-info` (see if kubectl is connected to a cluster)
  - also: `kubectl config view` (get info from `~/.kube/config`)
- `kubectl get pods -A` (see all pods in all namespaces)
- `kubectl get all -n default` (see all resources in the default namespace)
- `kubectl delete all -l app=<name> -n default` (delete all resources with the label `app=<name>` in the default namespace)
- `kubectl get svc`
- `kubectl get pods --show-labels`
- `kubectl describe <type> <name>`
  - e.g. `kubectl describe pod my-pod`
- `kubectl delete <type> <name>`
- `kubectl get ep` (get endpoints)
- `kubectl explain <name-of-field>`
  - e.g. `kubectl explain Pod.spec.containers`
  - use optional `--recursive` flag for more details.
- `kubectl edit <type> <name>` (lets you inline edit deployed resources)
  - e.g. `kubectl edit sc standard`
- `kubectl exec -it some-pod /bin/bash` (access the bash shell for some-pod)
  - alternatively, `kubectl exec some-pod -- <bash-command>` to run a single command without opening the shell.
- `kubectl logs <pod-name>`
  - specify container: `kubectl logs <pod-name> -c <container-name>`

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
- etcd store:
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

## Data Plane Nodes (Worker Nodes)

The data plane consists of the worker nodes that actually run your containerized workloads.

services that make up the data plane:

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
Everything in kubernetes gets labels. The service object looks for pods that share all of the same labels. If a pod has the same labels as the service object, plus a few extra, it will still be included in the service object scope. If a pod contains one but not all labels, it won't be included.

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

to rollback, check the replica sets and note the hashes:

```
kubectl get rs
```

check the revision details to find the hash:

```
kubectl rollout history deploy web-deploy --revision=<number>
```

rollback to the desired revision:

```
kubectl rollout undo deploy web-deploy --to-revision <number>
```

a rollback is simply a rollout in reverse.

## Using Storage

the Kubernetes Persistent Volume Subsystem decouples data from application pods. Pods access the volume, but if the pod or cluster is stopped or deleted, the volume remains. This means that the volume does live within the cluster itself.

External Storage --> plugin to the cluster --> PersistentVolume (pv) --> PersistentVolumeClaim (pvc) --> your application pod

pvc's enforce exclusive access from a pod to the pv. No other pods can connect.

one pv is needed per external storage volume. One pvc is needed per pv. The volume size in each yaml file needs to match.

StorageClasses allow this process to be dynamic.

Container Storage Interface (CSI): storage plugin code living outside of the main kubernetes code tree. the plugin will vary based on where the storage is being hosted. Cloud storage usually have their own dedicated plugins, but there are on-prem options as well.

pv example:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ps-pv
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ps-fast
  capacity:
    storage: 50Gi
  persistentVolumeReclaimPolicy: Retain # this pv will NOT automatically delete the underlying storage volume once the associated pvc is deleted
  gcePersistentDisk: # name of the plugin for Google Cloud
    pdName: ps-vol # name of the persistent disk on Google Cloud
```

`accessModes`:

- Read-write once (RWO)
  - this pv can be claimed once in read-write mode by one pod (pod can contain multiple containers though).
- Read-write many (RWM)
  - read-write mode, can be claimed by more than one pod.
- Read-only many (ROM)
  - read-only mode, can be claimed by more than one pod.

pvc example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ps-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ps-fast
  resources:
    requests:
      storage: 50Gi
```

note that the pvc yaml looks about identical to the pv yaml, minus the CSI plugin.

add the pv and pvc to the cluster:

```sh
kubectl apply -f ps-pv.yml
kubectl get pv ps-pv
kubectl apply -f ps-pvc.yml
kubectl get pvc ps-pvc
kubectl get pv ps-pv
```

note the ps-pv STATUS goes from `Available` to `Bound` after adding the pvc.

example pod referencing the pvc:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: first-pod
spec:
  volumes:
    - name: fast50g # volume name
      persistentVolumeClaim:
        claimName: ps-pvc # pvc reference
  containers:
    - image: ubuntu:latest
      name: ctr1
      command:
        - /bin/bash
        - "-c"
        - "sleep 60m"
      volumeMounts:
        - mountPath: /data
          name: fast50g # volume to be mounted to this container
```

### Dynamic Provisioning

StorageClasses (sc) prevent the need to explicitly create pv's; when `volumeBindingMode: WaitForFirstConsumer` is defined on the sc, and when a pvc exists, if a pod is deployed that references the pvc, the corresponding pv will be spun up automatically.

StorageClass:

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: ps-gcp-fast
  annotations:
    storageclass.kubernetes.io/is-default-class: "true" # any pvc that doesn't explicitly request a storage class will get this one
provisioner: kubernetes.io/gce-pd # plugin (Google Compute Engine Persistent Disk)
volumeBindingMode: WaitForFirstConsumer # hold off creating the storage volume and pv until a pod starts that uses it (you can still create a pvc beforehand)
parameters: # values here are dependent on the plugin
  type: pd-ssd
  replication-type: none
```

create:

```
kubectl apply -f ps-sc.yml
kubectl get sc
```

larger example:

- pvc
- 2 containers in a single pod:
  - nginx webpage, volume used for html directory.
  - ubuntu (mounting the same volume).
- service to expose it to the internet

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-htmlvol
spec:
  storageClassName: "ps-gcp-fast"
  accessModes:
    - ReadWriteOnce # only one pod can access
  resources:
    requests:
      storage: 25Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: sc-pod
  labels:
    app: stg
spec:
  volumes:
    - name: htmlvol
      persistentVolumeClaim:
        claimName: pvc-htmlvol
  containers: # 2 containers
    - name: main-ctr
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: htmlvol
    - name: helper-ctr
      image: ubuntu
      command:
        - /bin/bash
        - "-c"
        - "sleep 60m"
      volumeMounts:
        - mountPath: /data
          name: htmlvol
---
apiVersion: v1
kind: Service # expose to internet
metadata:
  name: lb
spec:
  selector:
    app: stg
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

if the above sc is in the cluster, and the above ps-scpod.yml file is deployed, a pv resource will be created automatically.

inspect the pod:

```
kubectl describe pod sc-pod
```

## Multi-Container Pod Use Cases

here are a few patterns for running multiple containers in a single pod:

- the init pattern
  - one container starts, then once condition is met it spins down and the next container starts.
- the sidecar pattern
  - sidecar container starts just before and runs in parallel with main app container.
- the adapter pattern
  - variation on sidecar: helper container that transforms data from the main app container, e.g. take log output from the main app and standardize formatting for external service. Both the main app and the sidecar typically share some sort of storage.
- the ambassador pattern
  - variation on sidecar: The sidecar acts as a proxy to forward information. The main app usually sends info to a port that the sidecar is listening to, and the sidecar forwards it outside of the pod.

recall that containers sharing a pod share resources and are deployed together as one unit.

init pattern:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ps-init
  labels:
    app: initializer
spec:
  initContainers: # init pattern defined here
    - name: init-ctr # this container runs first
      image: busybox
      command:
        [
          "sh",
          "-c",
          "until nslookup pluralsight-ftw; do echo waiting for pluralsight-ftw service; sleep 1; done; echo Service found!",
        ]
  containers: # once condition is met, previous container stops and this one starts
    - name: web-ctr
      image: nigelpoulton/web-app:1.0
      ports:
        - containerPort: 8080
```

note that the `initContainers` is an array whose containers will spun/down in the order that they are defined in.

sidecar pattern:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: alpine:latest
          command:
            [
              "sh",
              "-c",
              'while true; do echo "logging" >> /opt/logs.txt; sleep 1; done',
            ]
          volumeMounts:
            - name: data
              mountPath: /opt
      initContainers:
        - name: logshipper
          image: alpine:latest
          restartPolicy: Always # If an init container is created with its restartPolicy set to Always, it will start and remain running during the entire life of the Pod
          command: ["sh", "-c", "tail -F /opt/logs.txt"]
          volumeMounts:
            - name: data
              mountPath: /opt
      volumes:
        - name: data
          emptyDir: {}
```

adapter pattern:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
spec:
  containers:
    - name: web-ctr
      image: nigelpoulton/nginxadapter:1.0
      ports:
        - containerPort: 80
  initContainers:
    - name: transformer
      image: nginx/nginx-prometheus-exporter
      restartPolicy: Always
      args: ["-nginx.scrape-uri", "http://localhost/nginx_status"]
      ports:
        - containerPort: 9113
```

note: I edited the example above for the new `restartPolicy: Always` syntax. I _think_ it's correct.

ambassador pattern:

- external api/service:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: external-app
  labels:
    app: ambassador
spec:
  containers:
    - name: nginx-outside
      image: nigelpoulton/nginx-outside:1.0
      ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: ps-ambassador
spec:
  selector:
    app: ambassador
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

- pod using ambassador pattern:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ambassador-app
spec:
  containers:
    - name: main-app
      image: nigelpoulton/curl:1.0
      command: ["sleep", "3600"]
    - name: ambassador
      image: nigelpoulton/nginx-ambassador:1.0
```

- the main-app is sending info to port 9000 (defined within the image itself).
- the ambassador is listening to port 9000 (defined within the image itself).
- the ambassador proxys the info to `server ps-ambassador` on port 80 (defined within the image itself).

## Securing Apps with Service Accounts

each namespace has its own service account. By default, each namespace gets a service account called "default". Every pod is associated with the "default" service account if `serviceAccountName` isn't specified.

the service account is what manages certs for authN/Z.

every kubectl command hits the cluster api server (which you can find with `kubectl cluster-info`, "Kubernetes control plane is running at \<ip>"). The api server inspects the cert provided by kubectl to verify it is coming from a legit source either signed by the cluster or certificate authority that the cluster trusts. The name of the user is then passed to the cluster's authorization plugin (usually an RBAC plugin).

this same authN/Z process happens from internal requests in the server that leave the cluster, except this time the package attempting to leave the cluster comes with a token that is associated with the pod's service account.

see the yaml `serviceAccountName` property on a pod to view the pod's service account:

```
kubectl get pod mypod -o yaml
```

see the service accounts:

```
kubectl get sa
```

view the sa token (`Tokens` property):

```
kubectl describe sa default
```

view token details:

```
kubectl get secret <token-name>
kubectl describe secret <token-name>
```

Role and RoleBinding example:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: svc-ro
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["services"]
    verbs: ["get", "watch", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: svc-ro
  namespace: default
subjects: # assign this role to a service account called "service-reader"
  - kind: ServiceAccount
    name: "service-reader"
    namespace: default
roleRef:
  kind: Role
  name: svc-ro
  apiGroup: rbac.authorization.k8s.io
```

the above yaml creates a role that is only allowed to get, watch, or list services. This role will be bound to a sa called "service-reader", which we will create in a minute.

```
kubectl apply -f rbac.yml
kubectl describe role svc-ro
kubectl get clusterrolebinding
```

The `docker-for-desktop-binding` clusterrolebinding makes all service accounts admin on the cluster. this effectively disables rbac, so for the demo it should be deleted:

```
kubectl delete clusterrolebinding docker-for-desktop-binding
```

create the "service-reader" service account. The role binding created earlier is applied automatically:

```
kubectl create serviceaccount service-reader
```

check that the certificates were in fact added (in the `Tokens` field):

```
kubectl describe sa server-reader
```

now add the pod (ambassador pattern):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sa-demo
spec:
  terminationGracePeriodSeconds: 1
  serviceAccountName: service-reader
  containers:
    - name: app
      image: nigelpoulton/curl:1.0
      command: ["sh", "-c", "sleep 9999"]
    - name: test1
      image: nigelpoulton/k8s-api-proxy:1.0
```

```
kubectl apply -f pod2.yml
kubectl exec -it sa-demo bash
```

note that for the exec command, if a container is not specified than it will attach to the first one defined in the yaml file.

in the bash shell, the main app is able to call `curl localhost:8001/api/v1/namespaces/default/services/` because it has the role permissions to get, watch, or list services. If another resource is specified such as `curl localhost:8001/api/v1/namespaces/default/pods/`, a 403 forbidden error is returned.

## Final Demo

Example for AWS Elastic Kubernetes Service (EKS):

```yaml
kind: StorageClass # dynamically provision storage from AWS Elastic Block Store
apiVersion: storage.k8s.io/v1
metadata:
  name: finale1-gcp-pd
provisioner: kubernetes.io/aws-ebs
volumeBindingMode: WaitForFirstConsumer
parameters:
  type: io1
  iopsPerGB: "10"
  fsType: ext4
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: reader
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pvc-ro
rules:
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "watch", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pvc-ro
  namespace: default
subjects:
  - kind: ServiceAccount
    name: "reader" # bind the pvc get/watch/list roles to the reader sa
    namespace: default
roleRef:
  kind: Role
  name: pvc-ro
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-finale-1
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: finale1-gcp-pd # matches the sc defined above
  resources:
    requests:
      storage: 25Gi
---
apiVersion: v1
kind: Service # not strictly needed for the app, but we can use it to connect to the app when it's running
metadata:
  name: finale-svc
spec:
  selector:
    app: finale-1
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
---
apiVersion: v1
kind: Pod
metadata:
  name: finale-1
  labels:
    app: finale-1 # label matches load balancer service
spec:
  serviceAccountName: reader # use above sa
  volumes:
    - name: pvc-finale-1 # use above pvc (which tells the sc to make a pv)
      #    emptyDir: {} # if running locally (not on cloud), uncomment this and comment out next two lines. You can also comment out the sc and pvc above.
      persistentVolumeClaim:
        claimName: pvc-finale-1
  securityContext: # needed because mounting external storage from some platforms can mess with permissions and stop some of our containers running later.
    fsGroup: 0
    runAsUser: 0
  initContainers:
    - name: init-pv # looks for pvc. It has permission to do so because the sa has get/watch/list pvc access. Once pvc is found, the container exits.
      image: nigelpoulton/k8s-api-proxy:1.0
      command: ["sh", "-c"]
      args:
        [
          "until kubectl get pvc pvc-finale-1; do echo waiting for PVC; sleep 1; done; echo PVC found!",
        ]
    - name: init-sync # sends a git repo to the shared volume, then exits.
      image: k8s.gcr.io/git-sync:v3.1.5
      volumeMounts:
        - name: pvc-finale-1 # mount to the pvc volume
          mountPath: "/tmp/git"
          readOnly: false
      env:
        - name: GIT_SYNC_REPO
          value: https://github.com/nigelpoulton/ps-sidecar.git # contains a single index.html file
        - name: GIT_SYNC_BRANCH
          value: master
        - name: GIT_SYNC_DEPTH
          value: "1"
        - name: GIT_SYNC_ROOT
          value: "/tmp/git"
        - name: GIT_SYNC_DEST
          value: "html"
        - name: GIT_SYNC_ONE_TIME
          value: "true"
    - name: init-svc # looks for the Service finale-svc, then exits.
      image: busybox
      command: [
          "sh",
          "-c",
          "until nslookup finale-svc; do echo waiting for finale-svc service; sleep 1; done; echo Service found!",
        ] # note that nslookup uses an external DNS mechanism rather than by asking kubernetes which is why the sa does not need RBAC access to the service.
  containers: # initialization is complete and the main app can start
    - name: app
      image: nginx
      ports:
        - containerPort: 80
      volumeMounts:
        - mountPath: "/usr/share/nginx"
          name: pvc-finale-1 # reference the shared volume that now contains the github repo. nginx can now serve the index.html file on the repo.
    - name: sidecar-sync # syncs any changes that occur to the gh repo.
      image: k8s.gcr.io/git-sync:v3.1.5
      volumeMounts:
        - name: pvc-finale-1
          mountPath: "/tmp"
      env:
        - name: GIT_SYNC_REPO
          value: https://github.com/nigelpoulton/ps-sidecar.git
        - name: GIT_SYNC_BRANCH
          value: master
        - name: GIT_SYNC_DEPTH
          value: "1"
        - name: GIT_SYNC_DEST
          value: "html"
```

## Learning Check

- Q: What best defines a raw block volume?
  - A: It's like an unformatted disk drive.
- Q: What is true about PVC-to-PV bindings?
  - A: They are exclusive.
- Q: How does a pod that does not explicitly define a service account authenticate with the Kubernetes API server?
  - A: Using the default service account from the default namespace.
- Q: What is true of the CSI?
  - A: It allows storage driver code to be implemented out-of-tree.
- Q: What is an objective of the Kubernetes persistent volume subsystem?
  - A: To decouple the lifecycle of storage from pods.
- Q: Which Kubernetes API object does a PodSpec need to reference in order to use a persistent volume?
  - A: Persistent volume claim (PVC).
- Q: Which use-case does an ambassador container help with?
  - A: Brokering network connections to external sources.
- Q: Which accessModes does Kubernetes support for access to volumes?
  - A: Read-write-once, read-write-many, and read-only-many.
- Q: Which authorization plug-in do most Kubernetes clusters enable by default?
  - A: RBAC.
- Q: What is the purpose of volumeBindingMode: WaitForFirstConsumer?
  - A: To hold off creating PVs and external assets until a Pod mounts them.
