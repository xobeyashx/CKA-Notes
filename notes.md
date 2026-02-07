# CKA Notes:

## DAY 1: 1/27 - Exploring Cluster. Accessing in a production. Containers vs Pods

Autocompletion:

```
source <(kubectl completion bash)
```

```echo "source <(kubectl completion bash)" >> ~/.bashrc```

`6443` is port for k8s

```crictl images``` = calico (container)

```kubectl get nodes -o wide```

```kubectl describe node node1```

labels:
sort resources and take action

annotations:
for humans

kubelet:
send info to *apiserver* in master node

in production environment, you would work in your machine with kubeadmin right/credentials, not in master node
you'll manage everything in your machine aka workstation
only thing needed is kube admin credentials (config file or login) RBAC (role-based access control)

docker --> creates container + IP

k8s--> creates pods --> pods create containers (ip/volume attached to pod) (cpu/memory attached to container)

pod is wrapping around container (pea pod example)

1 pod contains one container which runs one process
1 pod can have multiple containers but both containers will share same resources (ip/volume)


## DAY 2: 1/28 - Understanding kubernetes resources. Creating Pods imperative. Logs.


-kubectl api-resources
shows all resources

-kubectl get crd
custom resources installed

-kubectl get crd | grep -i traefik > /tmp/custom-resources.txt

-kubectl explain secrets
-kubectl explain pods

similar to man pages

-kubectl run webapp --image=docker.io/lovelearnlinux/webserver:v1
-kubectl get pods -o wide
-kubectl describe pod webapp

-kubectl delete pod <podname>

-kubectl get events
event you did in past - only stored past hour by default

-kubectl logs <podname>
container logs

-kubectl describe <podname>

CrashLoopBackOff -kubectl logs <podname>
MYSQL\_ROOT\_PASSWORD error

kubectl delete pod database
kubectl run database --env=MYSQL\_ROOT\_PASSWORD=redhat --image=docker.io/mysql:latest
kubectl get pods -o wide


## DAY 3: 1/29 -  Kubectl create vs kubectl apply. Labels and Selectors

-vi mypod.yaml

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

-kubectl create -f mypod.yaml  (only creates, does not update)

-kubectl describe pod webapp2

-kubectl explain pod
what to put in yaml file

-kubectl get pod webapp -o yaml
shows how its written in yaml (10 lines turns ot more in /etcd)

********
steps to creating and updating a pod:
-vi newpod.yaml
-kubectl create -f newpod.yaml

update yaml file with updates (ex: labels)
-kubectl apply -f newpod.yaml

deleting pod
-kubectl delete -f newpod.yaml
*********

can also create multiple resources using a single yaml file -- these are two pods but we can also create 2 containers in a single pod as well by adding another container name (like the second pod)

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

-kubectl apply -f mypod.yaml
when u delete using (kubectl delete -f yamlfile.yaml) it'll delete both 


## DAY 4: 2/2 - Quality of Service

cpu and memory

Quality of Service Classes: burst, guarantee

burst:

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



guarantee:

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


metric consumption - need to install (not native to k8s)
using top command

-kubectl top nodes
-kubectl top pods


kubelet monitors the limits (cpu and memory)
scheduler monitors requests (cpu and memory) 

kubelet updates the apiserver if nay server tries to exceed the limit
OOM (out of memory) kille rwill execute
container will restart (customers will complain about dropped connection)

most seasoned professionals will not put a (limits: cpu:) because that means we're not using what's available

its in Burstable mode if not defined cpu in limits

best effort- non-time sensitive jobs (report generating, background jobs, non-time sensitive jobs,etc..)
bustable- varying load patterns (webservers, api-services, etc...)
guarantee- apps that require high level resources (mission critical apps, video streaming, online gaming, trading, etc...)


## DAY 5: 2/3 - Deployment PART ONE


































