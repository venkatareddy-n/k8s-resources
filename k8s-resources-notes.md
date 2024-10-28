install kubectl on linux server
=========================
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.31.0/2024-09-12/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv kubectl /usr/local/bin/

kubectl version
Client Version: v1.31.0-eks-a737599
-----------------------------------

Resources
==========
namespace
----------
In Kubernetes, namespaces provide a mechanism for isolating groups of resources within a single cluster.
Names of resources need to be unique within a namespace, but not across namespaces. 

namespace structure
-------------------
apiVersion:
kind: Namespace
metadata:
  name:
  labels:
spec:

sample namespace creation
-------------------------
apiVersion: v1
kind: Namespace
metadata: 
  name: expense
  labels:
    project: expense-project
    environment: dev

---------------------------------
kubectl apply -f 01-namespace.yaml  --> to create namespace

kubectl get namespaces --> to check namespaces

kubectl delete namespace <namespace_name> --> to delete namespace

kubectl delete namespace expense
------------------------------------------------------------

PODs
====
Pods are the smallest deployable units in kubernetes, 
pod can contain one or many containers

A Pod is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers.

apiVersion: v1
kind: Pod
metadata:
  name: nginx-server
spec:
  containers:  # this is equals to docker run -d --name nginx-server nginx
  - name: nginx-server
    image: nginx


kubectl apply -f 02-pod.yaml --> create the bod

kubectl get pods --> to check the pods list

kubectl exec -it nginx-server -- bash  --> to enter inside the pod

kubectl delete pod <pod_name> --> to delete the pod
kubectl delete pod nginx-server
--------------------------------------

multi-container
===============
one pod have 2 containers 

kind: Pod
apiVersion: v1
metadata:
  name: multi-container
spec:
  containers:
  - name: nginx-server
    image: nginx
  - name: almalinux-server
    image: almalinux:9
    command: ["sleep","1000"]

Error
=====
CrashLoopBackOff --> with your command container not able to start, check the command to run container

RunContainer Error --> with your command container not able to start

kubectl exec -it multi-container -c almalinux-server -- bash --> enter to almalinux container

kubectl exec -it multi-container -c  nginx-server -- bash --> enter to nginx container
----------------------------------------------------------------------------------

LABELS
=======
kind: Pod
apiVersion: v1
metadata:
  name: labels
  labels:
    project: expense
    module: backend
    environment: dev
spec:
  containers:
  - name: nginx
    image: nginx

kubectl apply -f 04-labels.yaml --> to create pod

to check full information of the pod

kubectl describe pod <pod_name>
kubectl describe pod labels --> to check pod information in docker we use inspect
---------------------------------------------------------------------------------

Annotations
===========
Annotations we use urls, special commands
we can keep complex characters

kind: Pod
apiVersion: v1
metadata:
  name: annotations
  annotations:
    imageregistry: "https://hub.docker.com/"
    buildURL: "https://jenkins.joindevops.com/expense/backend/build/67"
spec:
  containers:
  - name: nginx
    image: nginx

kubectl describe pod <pod_name>
kubectl describe pod annotations --> to check pod information
-------------------------------------------------------------



environment variables
=====================
env in Dockerfile should rebuild if you change
env in manifest no need to rebuild, just restart is enough 

apiVersion: v1
kind: Pod
metadata:
  name: env-demo
spec:
  containers:
  - name: nginx
    image: nginx
    env:
    - name: course
      value: devops
    - name: owner
      value: "siva kumar"

kubectl describe pod env-demo

kubectl exec -it env-demo -- bash --> to enter inside the 
---------------------------------

Resources Limits
================
apiVersion: v1
kind: Pod
metadata:
  name: limits
spec:
  containers:
  - name: nginx-server
    image: nginx
    resources:
      requests:
        memory: 68Mi
        cpu: 100m        
      limits:
        memory: 100Mi
        cpu: 120m
        
kubectl apply -f 07-resource-limits.yaml

kubectl get pods

kubectl describe pod <pod_name>

kubectl describe pod limits

output
------
Limits:
      cpu:     120m
      memory:  100Mi
    Requests:
      cpu:        100m
      memory:     68Mi
---------------------------

Config map
==========
to store the variables and values

apiVersion: v1
kind: ConfigMap
metadata:
  name: config-map-info
data:
  course: devops
  trainser: "siva kumar"
  duration: "4 Months"

kubectl apply -f 08-config-map-info.yaml

kubectl get configmaps --> t0 see all config maps

kubectl describe configmap config-map-info --> configmap describe to see variables

output
------
[ ec2-user@ip-172-31-89-76 ~/k8s-resources ]$ kubectl describe configmap config-map-info
Name:         config-map-info
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
course:
----
devops

duration:
----
4 Months

trainser:
----
siva kumar

---------------

apiVersion: v1
kind: Pod
metadata:
  name: pod-config
spec:
  containers:
  - name: nginx-server
    image: nginx
    env:
      - name: course_name
        valueFrom:
          configMapKeyRef:
            name: config-map-info  # name of the config map, it will get values from there
            key: course  # env.name and config map key name no need to be same
      - name: trainser
        valueFrom:
          configMapKeyRef:
            name: config-map-info
            key: trainser 
      - name: total_duration
        valueFrom:
          configMapKeyRef:
            name: config-map-info
            key: duration 

kubectl apply -f 09-config-map-pod.yaml

kubectl exec -it pod-config -- bash --> enter inside the pod and check env

exit

kubectl edit configmap config-map-info  --> edit the values in config map and save

to effect the changes need to restart the pod (restart means delete the pod and create again)

kubectl delete pod pod-config --> delete the pod 

kubectl apply -f 09-config-map-pod.yaml

kubectl exec -it pod-config -- bash --> enter inside the pod and check env

exit
------------------
2nd method to get all values at a time

apiVersion: v1
kind: Pod
metadata:
  name: pod-config
spec:
  containers:
  - name: nginx-server
    image: nginx
    envFrom:
    - configMapRef:
        name: config-map-info   

-------------------------------------



