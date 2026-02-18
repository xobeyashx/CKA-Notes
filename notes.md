# CKA Notes:

## DAY 1: 1/27 - Exploring Cluster. Accessing in a production. Containers vs Pods

Autocompletion: `source <(kubectl completion bash)`

`echo "source <(kubectl completion bash)" >> ~/.bashrc`

`6443` is port for k8s

`crictl images` = calico (container)

`kubectl get nodes -o wide`

`kubectl describe node node1`

**labels**:
sort resources and take action

**annotations**:
for humans

**kubelet**:
send info to *apiserver* in master node

in production environment, you would work in your machine with **kubeadmin** right/credentials, not in master node
you'll manage everything in your machine aka workstation
only thing needed is kube admin credentials (config file or login) RBAC (role-based access control)

docker --> creates container + IP

k8s--> creates pods --> pods create containers (ip/volume attached to pod) (cpu/memory attached to container)

pod is wrapping around container (pea pod example)

1 pod contains one container which runs one process
1 pod can have multiple containers but both containers will share same resources (ip/volume)

## DAY 2: 1/28 - Understanding kubernetes resources. Creating Pods imperative. Logs.

`kubectl api-resources` = shows all resources

`kubectl get crd` = custom resources installed

`kubectl get crd | grep -i traefik > /tmp/custom-resources.txt`

`kubectl explain secrets`

`kubectl explain pods` = simalar to man pages

`kubectl run webapp --image=docker.io/lovelearnlinux/webserver:v1`

`kubectl get pods -o wide` = more info on pod

`kubectl describe pod webapp`

`kubectl delete pod <podname>` = delete pod

`kubectl get events` = event you did in past - only stored past hour by default

`kubectl logs <podname>` = container logs

`kubectl describe <podname>`

CrashLoopBackOff `kubectl logs <podname>`

MYSQL\_ROOT\_PASSWORD error

`kubectl delete pod database`

`kubectl run database --env=MYSQL\_ROOT\_PASSWORD=redhat --image=docker.io/mysql:latest`

`kubectl get pods -o wide`

## DAY 3: 1/29 -  Kubectl create vs kubectl apply. Labels and Selectors

`vi mypod.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

`kubectl create -f mypod.yaml` (only creates, does not update)

`kubectl describe pod webapp2`

`kubectl explain pod`

what to put in yaml file

`kubectl get pod webapp -o yaml`
shows how its written in yaml (10 lines turns ot more in /etcd)

---

steps to creating and updating a pod:

`vi newpod.yaml`

`kubectl create -f newpod.yaml`

update yaml file with updates (ex: labels)
`kubectl apply -f newpod.yaml`

deleting pod
`kubectl delete -f newpod.yaml`

---

can also create multiple resources using a single yaml file -- these are two pods but we can also create 2 containers in a single pod as well by adding another container name (like the second pod)

```
apiVersion: v1
kind: Pod
metadata:
  name: webapp2
  labels:
    client: abcd
spec:
  containers:
  - name: boxone
    image: nginx:1.14.2
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: dbapp2
  labels:
    client: xyz
spec:
  containers:
  - name: boxone
    image: docker.io/library/httpd:latest
    ports:
    - containerPort: 80
  - name: boxtwo
    image: redis 
```

`kubectl apply -f mypod.yaml`
when u delete using (`kubectl delete -f yamlfile.yaml`) it'll delete both

## DAY 4: 2/2 - Quality of Service

cpu and memory

Quality of Service Classes: burst, guarantee

burst:

```
apiVersion: v1
kind: Pod
metadata:
  name: bustapp
  labels:
    client: abcd
    comnponents: frontend
    env: prod
    instance: jan2026
    part-of: employee
    version: "2.3"
spec:
  containers:
  - name: boxone
    image: nginx:1.14.2
    ports:
    - containerPort: 80
    resources:
      limits:
        cpu: 300m
        memory: 256Mi
      requests:
        cpu: 100m
        memory: 128Mi
```

guarantee:

```
apiVersion: v1
kind: Pod
metadata:
  name: guaranteeapp
  labels:
    client: abcd
    comnponents: frontend
    env: prod
    instance: jan2026
    part-of: employee
    version: "2.3"
spec:
  containers:
  - name: boxone
    image: nginx:1.14.2
    ports:
    - containerPort: 80
    resources:
      limits:
        cpu: 300m
        memory: 256Mi
      requests:
        cpu: 300m
        memory: 256Mi
```

metric consumption - need to install (not native to k8s)
using top command

`kubectl top nodes`

`kubectl top pods`

**kubelet** monitors the **limits** (cpu and memory)
**scheduler** monitors **requests** (cpu and memory)

kubelet updates the apiserver if any server tries to exceed the limit
OOM (out of memory) killer will execute
container will restart (customers will complain about dropped connection)

most seasoned professionals will not put a (limits: cpu:) because that means we're not using what's available

its in Burstable mode if not defined cpu in limits

best effort- non-time sensitive jobs (report generating, background jobs, non-time sensitive jobs,etc..)
bustable- varying load patterns (webservers, api-services, etc...)
guarantee- apps that require high level resources (mission critical apps, video streaming, online gaming, trading, etc...)

## DAY 5: 2/3 - Deployment PART ONE

```
apiVersion: apps/v1             #resource type
kind: Deployment                #resource type
metadata:                           #name +labels
  name: nginx-deployment            #name +labels
  labels:                           #name +labels
    app: nginx                      #name +labels
spec:                 
  replicas: 3                   #replica sets  -  3 pods with app=nginx
  selector:                     #replica sets
    matchLabels:                #replica sets
      app: nginx                #replica sets
  template:                   
    metadata:                                   #template - how the app will be created
      labels:                                    #template - how the app will be created
        app: nginx                               #template - how the app will be created
    spec:                                         #template - how the app will be created   
      containers:                                #template - how the app will be created
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        resources:                               #template - how the app will be created
          limits:                                     
            cpu: 200m
            memory: 256Mi
          requests:
            cpu: 100m                                #template - how the app will be created
            memory: 128Mi                                     
```

`metadata label app` can be different

`spec matchLabels app` must match `template label app`

`kubectl get pods`

deploymentname-replicasetname-podname = `nginx-deployment-8b7fcb74-6nbdk`

a Deployment manages ReplicaSets, and those ReplicaSets create and manage Pods

so a replicaset will create and manage pods. if one pod is deleted, it will create another one with a different ip. but the **service** has a fixed ip which the customer/user will use/need. if we have a 100 pods, we will not need 100 ips as we will only need one fixed ip from the service.

**Label of Pod** must match - selector of replicaset and selector of service

**containerPort** must match - **targetPort**

## DAY 6: 2/4 - Deployment PART TWO

**NodePort**

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc-np
spec:
  type: NodePort                #what makes it NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      # By default and for convenience, the `targetPort` is set to
      # the same value as the `port` field.
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane
      # will allocate a port from a range (default: 30000-32767)
      #     nodePort: 30007 (optional)
```

`kubectl get svc` = you will now see NodePort

ClusterIP is internal-only, NodePort exposes a service on every node, and both route traffic to Pods via kube-proxy.

**NodePort** exposes the service on every node's IP - not recommended in production set-up

`APP -> Deployment -> ReplicaSets -> Pods <- Service`

ReplicaSets = Manual Scaling and AutoScaling (hpa)

**Troubleshooting:**

Pod is in pending state -> edit deployment and lower memory limit and request.

Manual Scaling: `kubectl scale deployment webapp --replicas 6`

Auto Scaling or **HPA** (horizontal pod autoscaling)

* Name of Deployment
* When to Scale? CPU > 80%
* How to Scale? scaling policy

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
spec:
  behavior:
  scaleDown:
    stabilizationWindowSeconds: 20
    policies:
    - type: Pods
      value: 1
      periodSeconds: 15
  scaleUp:
    stabilizationWindowSeconds: 30
    policies:
    - type: Pods
      value: 1
      periodSeconds: 30
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 10
```

as load goes up, autoscaling will keep creating pods until it reaches the MAX defined pods `(maxReplicas: 10)`

will only do **cpu** and **memory**

**KEDA - Kubernetes Event Driven Autoscaling (higher level hpa)**

* can integrate with Prometheus and other tools for higher level use

## DAY 7: 2/5 - Deployment PART THREE

recreate - ramped - blue/green - canary - a/b testing

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-deploy
  annotations:
    kubernetes.io/change-cause: "changing version to new"
spec:
  revisionHistoryLimit: 5
  replicas: 10
  selector:
     matchLabels:
        app: hello-world
  minReadySeconds: 10
  strategy:
     type: RollingUpdate
     rollingUpdate:
        maxUnavailable: 1
        maxSurge: 1
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-pod
        image: lovelearnlinux/webserver:v1    #version
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 100m
            memory: 128Mi
          requests:
            cpu: 50m
            memory: 100Mi
        readinessProbe:
          exec:
            command:
            - cat
            - /var/www/html/index.html
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 4
          failureThreshold: 2
          successThreshold: 1
        livenessProbe:
          exec:
            command:
            - cat
            - /var/www/html/index.html
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 4
          failureThreshold: 6
          successThreshold: 1

# kubectl set image deploy/hello-deploy hello-pod=lovelearnlinux/webserver:v2 --annotation=update image from v1 to v2
# kubectl annotate deployments.apps hello-deploy kubernetes.io/change-cause="image changed to nginx:latest"
# kubectl rollout history deployment hello-deploy --revision 5


```

`revisionHistoryLimit: 5` - last 5 versions to keep in /etcd

this is to update/upgrade

## DAY 8: 2/9 - Probes

Find out if your application has become malfunctioned

* When you define a startupProbe:
* Kubernetes runs only the startupProbe first
* If it fails → container is restarted
* If it succeeds → Kubernetes enables:
  * livenessProbe
  * readinessProbe

So it acts like a gatekeeper before other probes kick in.

* so lets say we're running a application in 3 pods, kubernetes will not know if its working (in the case of `index.html` deleted as an example in one pod)
* it will work on 2 pods, but if it connects to the 3rd one (without `index.html`) it won't work
* pod will still show running, doesn't mean it's working
* this is where the `probes` come in (checks/probes the application)

`startupProbe` in Kubernetes is used to determine whether a container has successfully started before other health checks (like liveness and readiness probes) begin.

We have 3 probes:

* start-up probes -
  * checks if container is ready to serve
  * ready only once
  * thresholds/parameters:
    * `initialDelaySeconds: 5` - checks conditions after 5 seconds
    * `period: 5`
    * `waitTime: 2`
    * `failureThreshold: 3`
    * `success: 1`
* readiness probe
  * monitors pod throughout the life of pod - after startup
  * if fails, pod will be removed from service endpoint (quarantined)
  * timer starts after success of startup probe (if defined)
  * same parameters can be made as startup
    * `failureThreshold: 3` - 3 subsequent failures, pod will be removed/quarantined
* liveness probe
  * runs throughout life of pod
  * if failed, it will restart container - trying to recover from the failed state
  * timer will start after success of startup probe (if defined)
  * same parameters (or whatever you set)
    * `failureThreshold: 3` - 3 subsequent failures, container will restart

startupProbe: (under spec: -> - containerPort:)

```
startupProbe:
  exec:
    command:
    - cat
    - /usr/share/nginx/html/index.html
  failureThreshold: 5
  periodSeconds: 10
  initialDelaySeconds: 10
```

you can add the same with `readinessProbe` and `livenessProbe` and configure them to custom parameters for them to work after the `startupProbe`

[kubernetes docs on liveness-readiness-startup-probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

## DAY 9: 2/10 - Namespaces. Resource Quota. Limit Range

namespace is the equivalent of projects in openshift

A namespace in Kubernetes is a logical partition of a cluster used to organize, isolate, and manage resources.

Think of it like folders inside one big filesystem (the cluster).

`kubectl get namespaces` or `ns`

`kubectl create ns dev`

```
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
    name: development
```

`kubectl config current-context`

`kubectl config get-contexts`

`kubectl config set-context --current --namespace=dev` - set namespace

```
metadata:
  name: development
  namespace: dev
```

kubectl apply -f mypod.yaml -n dev

**QUOTAS**

A Kubernetes quota is a way to put hard limits on how much cluster resources a team, app, or namespace is allowed to use.

Think of a quota like a budget for a namespace.

```
kubectl create quota my-quota
--hard=cpu=1,memory=1G,pods=2,services=3,replicationcontrollers=2,resourcequotas=1,secrets=5,persistentvolumeclaims=10
```

once you create a quota in a project, you cannot create a project without hard-coding (adding resource block) in the yaml file

**LimitRange**

sets limits on per pod basis - how much a single pod is allowed to consume in a cluster

apply LimitRange to default namespace, it will be applied in the default resource block (for those without resource block)

```
apiVersion: v1
kind: LimitRange
metadata:
  name: prod-limit-range
  namespace: prod
spec:
  limits:
  - default:                           # this section defines default limits
      cpu: 500m
      memory: 128Mi
    defaultRequest:                   # this section defines default requests
      cpu: 500m
      memory: 71Mi

    max:                              # max and min define the limit range
      cpu: "300m"
      memory: 256Mi
    min:
      cpu: 100m
      memory: 128Mi
    type: Container
```

## DAY 10: 2/11 - Network Policies

Network Policies in Kubernetes control which pods can talk to which other pods (and external endpoints) — basically acting like a firewall inside your cluster.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
 name: prod-network-policy
 namespace: prod
spec:
 podSelector: {}      #{} mean all pods
  # matchLabels:
  #   role: db
 policyTypes:
 - Ingress               #incoming
 # - Egress              #outgoing
 ingress:
 - from:
    # - ipBlock:                            # ip can change so shouldn't reply on ip for network policies
     #    cidr: 172.17.0.0/16                # everything under -from is OR (ip or namespace or selector)
      #   except:
       #  - 172.17.1.0/24
   - namespaceSelector:
       matchLabels:
         project: myproject                   # namespace label in pod (can be custom if changed)  
     # podSelector:
     #  matchLabels:
     #     role: frontend
   ports:
   - protocol: TCP
     port: 80

```

if you want to only connect to a specific pod in an enviornment, make sure to set pod label as well

```
       podSelector:
       matchLabels:
        app: frontend
```

check with

```kubectl -n dev exec devtwo -- curl <ip>```

dev = namespace
devtwo  = podname in the dev namespace

multiple namespaces:

```
    - namespaceSelector:
        matchExpressions:
        - key: namespace
          operator: In
          values: ["frontend", "backend"]
```

## DAY 11: 2/12 - node selector. Affinity. Taints and Tolerations

`NodeSelector`, `Affinity`, and `Taints/Tolerations` are Kubernetes scheduling rules that control which nodes your Pods can (or cannot) run on.

The Kubernetes scheduler watches for unscheduled Pods and assigns them to the best possible node based on constraints, resources, and scoring rules.

so it will assign a pod to the node with the most available space (free space)

**specifying a node to go to:**

```
spec:
  nodeName: nodetwo
```

* complete app will go to nodetwo. usually it will distribute but this will onyl make it so it goes to one specific node
* it skips scheduler part when specifying a node to go to

#### NodeSelector

* label node based on their feature
* label nodes first (disk: ssd, cpu: i7/disk: sats, cpu: i5/disk: ssd, cpu: xeon/etc...)
* checked during scheduling (when getting deployed) & ignored during execution
* only single condition (affinity for multiple)

```
spec:
  nodeSelector:
    disktype: ssd
    cpu: xeon
```
above application will only go to nodes with labels ssd and xeon

`kubectl label node node1 disktype=ssd` can do same for cpu or any other lables

`kubectl label node node1 disktype=sata --overwrite` to change a label 

#### Affinity and anti-affinity

* can use multiple conditions
* same as NodeSelector with extra features

If no matching node exists → Pod stays Pending:
```
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In            # can put NotIn (not in ssd and sata)
            values:                 # OR
            - ssd
            - sata           

```

there is also `podAffinity` and `podAntiAffinity` as well. same concept. Controls scheduling relative to other Pods.

#### Taints and Tolerations

* NodeSelector & Affinity = Pod chooses node
* Taints = Node repels Pods

* Taints - Node
* Toleration - Pods

conditions on nodes (when a pod is trying to get deployed on a node, it needs to sepcifically have a label that allows it to be deployed to the node)

* key=value:policy

* `kubectl taint nodes node1 dedicated=prod:NoSchedule`

so if there is already pods running on the nodes, and you add a `taint`, those pods without the taint label will move to other pods (depends on the policy/ex: NoExecute)

## DAY 12: 2/16 - Secrets & Configmaps
