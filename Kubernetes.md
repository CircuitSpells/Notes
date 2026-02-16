# Kubernetes

- [k8s installation guide](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/).
- example code:
  - https://github.com/nigelpoulton/getting-started-k8s
  - https://github.com/nigelpoulton/ps-vols-and-pods
  - https://github.com/nigelpoulton/ckad
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
  - can also add the `-o yaml` flag to output the yaml
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
- `kubectl config set-context --current --namespace=<namespace-name>` (switch namespaces. see available namespaces with `kubectl get namespace`)
- `kubectl port-forward <pod-name> <desired-port>:<pod-port> &` (port forward your localhost to a port the pod is listening on)

useful for ckad exam:

- `kubectl create deploy <deploy-name> --image=<image>:<tag> --dry-run=client -o yaml > <file-name>.yaml` (creates a new deployment file)
- `kubectl create -f <path-to-files>` (create all resources in the directory. Can also use `apply` insted.)
- `kubectl set selector svc <service-name> 'key=value'` (update a service to point at pods with a certain label)
- `kubectl scale deploy <deployment-name> --replicas=<num-replicas>` (update a deployment to a new number of replicas)
- `kubectl set image deploy <deployment-name> <image-name>` (update a deployment to use a different image)
- `KUBE_EDITOR="nano" kubectl edit <object-type> <object-name>` (update the yaml directly with nano. No need to apply afterwards since `edit` points to the file Kubernetes is directly referencing. This is useful if you imperatively created the services and do not have the yaml files in your file system)

## Control Plane Nodes

note: used to be called "masters".

do not to run user apps on control plane nodes.

per normal H/A standards, 3 control plane nodes is standard. Do not pick an even number; if there is a split in the network, the side that knows it has the majority nodes will arbitrarily pick a new leader. If there are an even number on each side, the nodes will deadlock and go into read-only mode.

on a hosted kubernetes platform, the control plane is hidden from you and is instead managed by the cloud provider.

leader: one control plane node makes active changes to the cluster at any one time. The others are called followers. If the leader goes down, they elect a new one.

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
  - assigns tasks to worker nodes: affinity/anti-affinity, constraints, taints, resources.

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
  - executes pods.
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

## Services

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
  name: ps-nodeport # the name here gets registered with the cluster DNS
spec:
  type: NodePort
  ports:
    - nodePort: 31111 # service port outside the cluster
      port: 80 # service port inside the cluster
      targetPort: 8080 # target port of pod inside the cluster
      protocol: TCP
  selector:
    app: web # will only hook to pods with matching labels
```

a note on `spec.type` above: there are three options:

1. `ClusterIP` (default): service is only available within the cluster.
2. `NodePort`: allows for access from outside the cluster.
3. `LoadBalancer`: allows for external access via your cloud provider's load balancer.

Request flow:

- there are multiple worker nodes listening publicly on port `31111`. A request hits that port.
- one of the worker nodes forwards this request to the service, which is listening on port `80` inside the cluster.
- the service forwards to a pod listening on port `8080`.

to deploy the svc-nodeport.yml file:

```
kubectl apply -f svc-nodeport.yml
kubectl describe svc ps-nodeport
```

describe is good for troubleshooting--verify `Endpoints` has a value; this shows a list of pods that the Service is connected to. If the list is empty, the service is not connected to any pods.

you can also check the endpoints directly with the following:

```
kubectl get ep
```

troubleshooting tip: make sure the port, labels, DNS, and namespace are correct on the Service definition.

note that if `targetPort` is not specified on the service, it will default to the value set in `port`.

### Create a Cloud Load Balancer Service

config file for load balancer service:

```yml
apiVersion: v1
kind: Service
metadata:
  name: ps-lb
spec:
  type: LoadBalancer
  ports: # LoadBalancer's create an NodePort under the hood, no need to define one.
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

## Deployments

Deployments hold replica sets which manage pods:

- deployment controller:
  - watches api server for new deployments.
  - implements deployments.
  - constantly compares observed state with desired state.
- replica sets:
  - replica count, self-healing, old versions

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

- `spec.replicas` defines the number of pods.
- `spec.selector.matchLabels` should match `spec.template.metadata.labels`. It is how the deployment knows which pods to work on during things like rolling updates. The label at `metadata.labels` is for the deployment itself.
- `spec.template.spec.containers.imagePullPolicy` set to `Always` means it will always pull from the registry instead of locally, which can protect against malicious attacks where someone puts an image with the same name locally.

apply the deployment:

```
kubectl apply -f deploy.yml
kubectl get deploy,rs,pods
```

this will have created 5 pods, all running the same container. The replica set name will be the name of the deployment followed by a hash of the pod spec.

### Self-Healing

use `kubectl get pods` and copy a pod name, then run:

```
kubectl delete pod <pod-name>
```

if you run `kubectl get pods` soon thereafter, you will notice there are still 5 pods due to the replica set self-healing.

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
  minReadySeconds: 5 # how long a new pod needs to be running before being considered healthy and the next pods are terminated/spun up. Default 0.
  progressDeadlineSeconds: 60 # num seconds to wait before considering the pod to be stalled. Default 600s.
  revisionHistoryLimit: 5 # num of replica sets that can be rolled back. So this can be rolled back to 5 versions ago. Default 10.
  strategy:
    type: RollingUpdate # will update the pods one at a time any time as opposed to all at once. Default RollingUpdate.
    rollingUpdate:
      maxUnavailable: 0 # how many pods below `spec.replicas` can we go. In this case, we cannot go below 10 pods. Default 25%.
      maxSurge: 1 # how many pods above `spec.replicas` can we go. In this case, we can go up to 11 pods. Default 25%.
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

```
kubectl create -f deploy.yml --save-config
kubectl annotate deployment <name> kubernetes.io/change-cause="<change-details>" --overwrite=true
```

the annotation isn't strictly required but makes it a lot easier to roll back if needed. To see annotations metadata under `metadata.annotations`:

```
kubectl get deploy <name> -o yaml
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

see history of what happened during the deployment:

```
kubectl describe deployment <name>
```

### Rollbacks

get status of a deployment (success, failure, updating, etc.):

```
kubectl rollout status -f <deployment>.yaml
```

get history of deployments:

```
kubectl rollout history
```

get history about a deployment, optionally add revision:

```
kubectl rollout history deployment <name> [--revision=<number>]
```

rollback a deployment, optionally add revision number otherwise defaults to going back to the previous revision:

```
kubectl rollout undo -f <deployment>.yaml [--to-revision=<number>]
```

## Blue/Green Deployments

blue/green deployments in Kubernetes require 3 services: the blue service, the green service, and the public service. The blue and green services will both be private, showing what both environments look like to devs and the business. the public service is how the public interfaces with the application.

say the public service is pointing towards pods app-v1, created by deployment deployment-v1. The blue service also points towards the app-v1 pods, aka it will look exactly the same as the public service. The green service points towards the app-v2 pods created by the deployment deployment-v2. When v2 of the app is ready, the public service switches over to point at the app-v2 pods, and the blue service can now be used for further development.

blue deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      env: blue
  template:
    metadata:
      labels:
        app: myapp
        env: blue
    spec:
      containers:
        - name: myapp
          image: myapp:1.0
          ports:
            - containerPort: 80
```

blue service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-blue
spec:
  type: LoadBalancer
  selector:
    app: myapp
    env: blue
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

green deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      env: green
  template:
    metadata:
      labels:
        app: myapp
        env: green
    spec:
      containers:
        - name: myapp
          image: myapp:2.0
          ports:
            - containerPort: 80
```

green service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-green
spec:
  type: LoadBalancer
  selector:
    app: myapp
    env: green
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

public service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-public
spec:
  type: LoadBalancer
  selector:
    app: myapp
    env: blue # change to green when ready
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

once the public service yaml has been updated to point towards the new environment, simply apply:

```
kubectl apply -f public-service.yaml
```

## Canary Deployments

canary deployments run in parallel to the stable prod deployment. A subset of traffic is routed to the canary in order to verify stability before rolling the changes out to all users.

say we have a deployment deploy-stable, with pods pod-stable, and deployment deploy-canary with pods pod-canary. There is a single service that routes traffic to both the stable pods as well as the canary pods.

service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-stable
  labels:
    app: myapp
spec:
  type: LoadBalancer
  selector:
    app: myapp # selects all pods
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

stable deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-stable
spec:
  replicas: 4
  selector:
    matchLabels:
      app: myapp
      track: stable
  template:
    metadata:
      labels:
        app: myapp # referenced by service
        track: stable
    spec:
      containers:
        - name: myapp
          image: myapp:1.0
          ports:
            - containerPort: 80
```

canary deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-canary
spec:
  replicas: 1 # note 1 instead of 4, meaning 20% of the traffic
  selector:
    matchLabels:
      app: myapp
      track: canary
  template:
    metadata:
      labels:
        app: myapp # referenced by service
        track: canary
    spec:
      containers:
        - name: myapp
          image: myapp:1.1
          ports:
            - containerPort: 80
```

the number of replicas in each deployment determines the percentage of traffic routed to each.

note that this will randomly assign users to a potentially different pod every time they refresh the page.

## Storage: PersistentVolume, PersistentVolumeClaim, StorageClass

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

StorageClasses (sc) prevent the need to explicitly create pv's; when `volumeBindingMode: WaitForFirstConsumer` is defined on the sc, and when a pvc exists, if a pod is deployed that references the pvc, the corresponding pv will be spun up automatically. It also typically guarantees that the volume will be spun up in the same zone or region as the pod.

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

note that if two containers share the same volume, then their mountPaths point to the same file system even if the mountPaths are different.

inspect the pod:

```
kubectl describe pod sc-pod
```

## Multi-Container Pod Use Cases

here are a few patterns for running multiple containers in a single pod:

- the init pattern
  - one container starts, then once condition is met it spins down and the next container starts. e.g. frontend requires backend to spin up first, so an init container pings the backend pod until it is ready, then the init container spins down and the frontend container spins up.
- the sidecar pattern
  - sidecar container starts just before and runs in parallel with main app container.
- the adapter pattern
  - variation on sidecar: helper container that transforms data from the main app container, e.g. take log output from the main app and standardize formatting for external service. Both the main app and the sidecar typically share storage.
- the ambassador pattern
  - variation on sidecar: The sidecar acts as a proxy to forward information. The main app usually sends info to a port that the sidecar is listening to, and the sidecar forwards it outside of the pod. e.g., the main app connects to a db using a standard localhost port. The ambassador listens on that port, and forwards to the port that the db is listening on.

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
provisioner: kubernetes.io/aws-ebs # based on AWS EBS
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

## NetworkPolicies

more info here: https://kubernetes.io/docs/concepts/services-networking/network-policies/

by default, the kubernetes network is wide open. The network _plugin_ implements network policies.

multiple policies can be set.

Traffic not in a policy is implicitly denied--all policies are "allow" rules. There is no way to deny traffic. Meaning if a NetworkPolicy is added to a pod, suddenly no traffic is allowed except that NetworkPolicy's whitelist.

NetworkPolicy example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: demo-netpol
  namespace: example # network policies only apply to a given namespace
spec:
  podSelector:
    matchLabels:
      project: ckad # match to pods with the label project=ckad
    policyTypes:
      - Ingress
      - Egress
    ingress:
      - from: # allow traffic from any pods with the label project=ckad, in the example namespace, on port 9000
          - podSelector:
              matchLabels:
                project: ckad
            namespaceSelector: # note the lack of a "-" prefix in the yaml here. That is because this is a logical AND. Adding the "-" before "namespaceSelector" would make this a separate, that is, a logical OR.
              matchLabels:
                kubernetes.io/metadata.name: example
        ports:
          - protocol: TCP
            port: 9000
    egress: # allow outgoing traffic on port 9000 to only pods on this network
      - to:
          - ipBlock:
              cidr: 10.0.0.0/24
        ports:
          - protocol: TCP
            port: 9000
```

`ipBlock` is only used for internal cluster networks.

allow all egress traffic on a namespace:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-egress
  namespace: example
spec:
  policyTypes:
    - Egress
  podSelector: {}
  egress:
    - {}
```

```
kubectl get netpol
```

## Ingress

recall that services (ClusterIP, NodePort, LoadBalancer) have a 1:1 port mapping. This can be costly when 20 cloud LoadBalancer services are needed. Ingress allows for 1:many through a single cloud load balancer. It sits underneath the load balancer and redirects traffic to the k8s (typically ClusterIP) service.

Example flow:

- example.com/abc is routed to the cloud's load balancer. The load balancer sends the request to Ingress which is able to then send it to the proper ClusterIP Service which is expecting calls at abc.example.com.
- example.com/xyz is routed to the cloud's load balancer. The load balancer sends the request to Ingress which is able to then send it to the proper ClusterIP Service which is expecting calls at xyz.example.com.

Ingress is only for HTTP/HTTPS.

Ingress uses a single LoadBalancer service on port 80 or 443. An Ingress is made of an object spec and and a controller; the object spec defines the rules and the controller implements them.

important: kubernetes does not ship with a native controller; you have to install one. Some cloud hosted options give you one out of the box. The controller is has the load balancer IP with the public IP address.

install the most common Ingress Controller for nginx (check latest controller version here: https://github.com/kubernetes/ingress-nginx/releases):

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.14.3/deploy/static/provider/cloud/deploy.yaml
```

installing the above controller will create a new namespace called `ingress-nginx`. Contained within that namespace, among other things, will be an ingressclass called ingress-nginx-controller.

```
kubectl get ingressclass -n ingress-nginx
```

note the `CONTROLLER` field contains `k8s.io/ingress-nginx`.

ingress classes can be used to install multiple controllers.

Ingress example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ckad
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx # name of the IngressClass that holds the Ingress controller
  rules:
    - host: www.app.com
      http:
        paths:
          - path: / # on root path of www.app.com
            pathType: Prefix
            backend:
              service:
                name: frontend # send to frontend ClusterIP Service
                port:
                  number: 80
    - host: dev.app.com
      http:
        paths:
          - path: / # on root path of dev.app.com
            pathType: Prefix
            backend:
              service:
                name: backend # send to backend ClusterIP Service
                port:
                  number: 80
    - host: app.com
      http:
        paths:
          - path: /abc # specifically on app.com/abc
            pathType: Prefix
            backend:
              service:
                name: frontend # send to frontend ClusterIP Service
                port:
                  number: 80
          - path: /xyz # specifically on app.com/xyz
            pathType: Prefix
            backend:
              service:
                name: backend # send to backend ClusterIP Service
                port:
                  number: 80
```

recall that ClusterIP Services each listen on their own IPs at a given port.

```
kubectl apply -f ingress.yaml
kubectl get ing
```

The `ADDRESS` field is the public IP of the load balancer that's been automatically created in the background.

view the routing rules:

```
kubectl describe ing <ingress-name>
```

## Jobs and CronJobs

Jobs run a specific number of pods through their lifecycle. Jobs are managed by the jobs controller (in the control plane). Jobs controller allows for restarts, retries, killing long-running processes, cleaning up, etc.

Jobs always try to ensure one or more Pods completes successfully.

Job example:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: ckad1
spec: # job
  completions: 5 # number of pods
  parallelism: 1 # number of pods to run at once
  backoffLimit: 4 # retry cap, uses exponential delay
  activeDeadlineSeconds: 90 # max duration of the pod lifetime. Pods then terminated. Takes precedence over backoffLimit.
  ttlSecondsAfterFinished: 90 # job and its resources are deleted after this many seconds. Can be set to 0. If unset, the job will not be cleaned up.
  template: # pod
    spec:
      restartPolicy: Never # other: OnFailure. App code needs to be able to handle local restarts. Setting to Never lets failed pods stick around so you can check their logs.
      containers: # container
        - name: ctr
          image: alpine:latest
          command: ["sh", "-c", 'echo "waiting..." && sleep 60']
```

CronJob example:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: ckad1-cron
spec: # cronjob
  schedule: "* * * * *"
  concurrencyPolicy: Allow # governs whether new jobs can start if previous Job instances are running. Options: Allow | Forbid | Replace
  startingDeadlineSeconds: 90 # if Job does not start this amount of time after the cron schedule, the Job is terminated. Setting to <10 means jobs might be missed.
  successfulJobsHistoryLimit: 5 # num successful jobs (and associated pods) to keep from previous runs. Default: 3.
  failedJobsHistoryLimit: 2 # num failed jobs (and associated pods) to keep from previous runs. Default: 1.
  jobTemplate: # job
    spec:
      template: # pod
        spec:
          restartPolicy: OnFailure
          containers: # container
            - name: ckad-container
              image: alpine:latest
              imagePullPolicy: IfNotPresent
              command:
                - /bin/sh
                - -c
                - echo "Hello world"; sleep 600
```

run:

```
kubectl apply -f job1.yml
kubectl get jobs
kubectl get pods
```

note that the pods are named after the job, and the pods will have labels like `controller-uid=<job-conroller-id>`.

show all job details including completion status, num of successful pods, and which pods are owned by the job:

```
kubectl describe job ckad1
```

deleting the job will delete the pods:

```
kubectl delete job ckad1
kubectl get pods
```

CronJobs run Jobs on a schedule.

cron format: `<minute> <hour> <day-of-month> <month> <day-of-week>`

e.g.:

- `0 4 * * 6` => 04:00 every Saturday
- `0 0 1 * *` => 00:00 (midnight) on the first day of every month
- `* * * * *` => every minute
- `*/2 * * * *` => every 2 minutes

important:

- CronJobs work on the timezone based on the k8s control plane API Server.
- CronJobs only wake up every 10 seconds and chekcs for tasks it missed.
- CronJobs only will create up to 100 missed tasks. This includes the time period defined in `startingDeadlineSeconds` (or 10 seconds if not defined). So if `startingDeadlineSeconds` is set to 120, only up to 100 Jobs will be created in that 2min period.

apply:

```
kubectl apply -f cronjob.yml
kubectl get cronjobs
kubectl get jobs
kubectl get pods
```

Jobs are named after the CronJob. Pods named after the Job.

Note that CronJobs only need the `schedule` parameter specified, the rest are optional:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: ckad1-cron
spec: # cronjob
  schedule: "* * * * *"
  jobTemplate: # ...
```

## Helm

install: https://helm.sh/docs/intro/install/

Helm is a package manager for Kubernetes. Helm installs charts, creating a new release for each install (not idempotent). Search Helm chart repositories to find new charts.

- helm client: cli for end users
- helm library: logic for executing operations

Helm Chart: used to create an instance of a k8s app. use charts to install, upgrade, and uninstall k8s apps.
Config: can be merged into a packaged chart to create a releasable object.

general workflow:

- `helm -h` - help
- `helm search hub` - view repos, defaults to Artifact Hub
- `helm repo add`
- `helm search repo`
- `heml show values` or `heml pull --untar` to dig into the files
- `helm values` - override defaults
- `helm install`, `helm upgrade`, `helm uninstall`
- `helm status`
- `helm list`

### Examples

search and install:

```sh
helm search hub nginx --list-repo-url -o yaml # look for app_version and repo url
helm repo add bitnami https://charts.bitnami.com/bitnami

helm repo list
helm search repo nginx # shows app version and chart version
helm show values bitnami/nginx

helm install my-nginx bitnami/nginx # installs with default template values

kubectl get all # show resources created by helm

helm list
helm uninstall my-nginx
```

untar and override default values with yaml:

```sh
helm pull --untar --version=15.0.9 bitnami/wordpress # adds a wordpress dir

cd wordpress
ls -l
cat Chart.yaml
cd ..
helm show values bitnami/wordpress --version=15.0.9 # shows values to override

nano wordpress-values.yaml # create new file with values:
# wordpressUsername: admin
# wordpressPassword: admin
# wordpressEmail: admin@admin.com
# wordpressFirstName: Jane
# wordpressLastName: Doe
# wordpressBlogName: admin.com
# service:
#   type: LoadBalancer

helm install my-wordpress bitnami/wordpress --values=wordpress-values.yaml -n dev --version=15.0.9 # add to the dev namespace

kubectl get all -n dev

helm uninstall my-wordpress -n dev
```

update repos, override default values inline, upgrade helm chart:

```sh
helm list -n dev
helm repo list
helm repo update # update all repos

helm search repo nginx --version=13.1.5 # find the pre-installed repo
helm show values bitnami/nginx --version=13.1.5 | grep replica # search for name of parameter to override

helm install my-app bitnami/nginx --version=13.1.5 -n dev --set replicaCount=5 # override default replicaCount

kubectl get pods -n devs # 5 pods created

helm upgrade my-app bitnami/nginx --version=13.1.8 -n dev # upgrade to 13.1.8
helm list -n dev # note the upgraded version number, and revision now equals 2

kubectl get pods -n dev

helm uninstall my-app -n dev
```

## Probes and Health Checks

Health Check: determine if app/service is functioning
Probe: diagnostic mechanism used by kubelet to determine container health

types of probes:

- startup:
  - verifies the app on the container has started.
  - runs at startup until the probe succeeds or fails, then stops running.
  - other probes are disabled until the startup probe succeeds.
  - used for containers that take a long time to start (databases, VMs).
- readiness:
  - determines if endpoints can receive requests.
  - runs during container's lifecycle.
  - if fails, removes pod from service.
- liveness:
  - determines if pod is healthy.
  - if fails, destroys pod and creates new one based on `restartPolicy`.
  - does not wait for readiness probes.
  - used for containers that can experience deadlocks or become unresponsive.

probes can return success, failure, or unknown (treated as failure, generally caused by a timeout).

handler options on pods for probes to check health:

- `httpGet` for web apps with a health endpoint
- `tcpSocket` for services with raw TCP endpoints
- `exec` for apps where state can only be verified internally

example with `httpGet`:

```yaml
# ...
spec:
  containers:
    - name: web-app
      image: nginx:latest
      readinessProbe:
        httpGet:
          path: /health
          scheme: http
          port: 80
        initialDelaySeconds: 2 # how long in seconds before probe runs
        periodSeconds: 5 # how often probe checks
        timeoutSeconds: 5 # how long probe will wait before timeout
        failureThreshold: 2 # how many times a probe can fail before a Failure condition occurs
        successThreshold: 2 # how many times a probe can succeed before a Success condition occurs
        terminationGracePeriodSeconds: 20 # time between requested shutdown and actual shutdown
```

check pod for health check failures:

```
kubectl describe pod my-pod
```

at the bottom will be `Events`. Health check successes won't log, but failures will with `Warning Unhealthy`.

troubleshooting:

- check probe configuration
- check events: `kubectl describe pod <pod-name>`
- check cluster events: `kubectl get events --sort-by='.lastTimestamp'`
- check logs: `kubectl logs <pod-name> [-c <container-name>]`

## ConfigMaps

ConfigMaps:

- store key-value pairs for configuration.
- require the pod to restart for changes to populate.
- are scoped to namespace.
- values can be multiline.
- values must be under 1MB.
- can be used as an argument, env var, or file.
- used for env config, feature flags, etc.

example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: production
  LOG_LEVEL: debug
binaryData:
  config.bin: | # base64
    U29tZSBiaW5hcnkgY29...
```

create imperatively:

```sh
kubectl create configmap app-config --from-literal=APP_MODE=production --from-literal=LOG_LEVEL=debug
```

imperative dump to yaml:

```sh
kubectl create configmap app-config --from-literal=APP_MODE=production --from-literal=LOG_LEVEL=debug --dry-run=client -o yaml > configmap.yaml
```

create from file:

```sh
kubectl create configmap app-config --from-file=startup.sh
```

view/describe:

```sh
kubectl get configmaps
kubectl describe configmap <name>
```

ref ConfigMap in Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
    - name: myapp
      image: busybox
      command: ["sh", "-c", "env"]
      envFrom: # full
        - configMapRef:
          name: app-config
```

or overwrite specific values:

```yaml
# ...
spec:
  containers:
    - name: myapp
      image: busybox
      command: ["sh", "-c", "env"]
      env: # partial
        - name: loglevel
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL
```

get config from a volume (file name is key, file contents is value):

```yaml
# ...
spec:
  containers:
    - name: myapp
      image: busybox
      command: ["sh", "-c", "env"]
      volumeMounts: # volume
        - name: config-volume
          mountPath: "/config"
          readOnly: true
      volumes:
        - name: config-volume
          configMap:
            name: app-config
```

## Secrets

Secrets:

- like ConfigMaps, are scoped to namespaces.
- can be passed into containers as arguments, env vars, or files.
- are stored unencrypted in etcd and are assessible by anyone with API access
  - if this is an issue, consider enabling encryption at rest
  - use restrictive RBAC policies

example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=
  password: UGE1NXcwcmQ=
```

`type` options:

- Opaque: arbitrary user-defined data
- kubernetes.io/service-account-token: service account token
- kubernetes.io/dockercfg: serialized ~/.dockercfg file data
- kubernetes.io/dockerconfigjson: serialized ~/.docker/config.json file data
- kubernetes.io/basic-auth: creds for basic auth
- kubernetes.io/ssh-auth: creds for ssh auth
- kubernetes.io/tls: data for a TLS client or server
- kubernetes.io/token: bootstrap token data

when using the `data` param, content must be pre-encoded:

```sh
echo -n "encode-this-password" | base64
```

otherwise the the `stringData` param and k8s will encode for you.

create secrets imperatively:

```sh
kubectl create secret tls tls-secret --cert=/cert/path --key=/key/path
```

view/describe:

```sh
kubectl get secret
kubectl describe secret db-secret
```

decode:

```sh
kubectl get secret db-secret -o jsonpath='{.data.username}' | base64 --decode
```

Secrets in Pods work just like ConfigMaps:

```yaml
# ...
spec:
  containers:
    - name: myapp
      image: busybox
      command: ["sh", "-c", "env"]
      envFrom: # full
        - secretRef:
          name: db-credentials
```

```yaml
# ...
spec:
  containers:
    - name: myapp
      image: busybox
      command: ["sh", "-c", "env"]
      env: # partial
        - name: DB_USER
          valueFrom:
            configMapKeyRef:
              name: db-credentials
              key: username
```

```yaml
# ...
spec:
  containers:
    - name: myapp
      image: busybox
      command: ["sh", "-c", "env"]
      volumeMounts: # volume
        - name: secret-volume
          mountPath: "/etc/secret"
          readOnly: true
      volumes:
        - name: secret-volume
          configMap:
            name: db-credentials
```
