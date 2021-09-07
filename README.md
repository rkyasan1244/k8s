# k8s
Basic Commands used fr CKAD:

https://github.com/mmumshad/kubernetes-training-answers
Lab:

https://uklabs.kodekloud.com/courses/labs-kubernetes-for-the-absolute-beginners-hands-on/

gain real time devops experience:
https://www.udemy.com/course/learn-kubernetes/learn/lecture/9997122#overview
https://www.youtube.com/watch?v=CnqjHRCZEt0

softwares reference:
https://www.udemy.com/course/learn-kubernetes/learn/lecture/9808148#overview

setup Kubernetes in local - 
https://www.udemy.com/course/learn-kubernetes/learn/lecture/9723230#overview





kubectl run webapp -> to run the contianer
kubectl cluster-info -> to get the cluster information
kubectl get nodes -> to get the nodes running in the cluster
kubectl run nginx --image=nginx -> start a pod
kubectl get pods --> returns pods information
kubectl describe pod nginx --> more info than get pods command
events information, which node the pod is running etc.,
kubectl get prod -o wide -> this give node where it is running, pod internal ip address.

pod.yml file:-
--------------------------------------------------------------------------------------------------
case sensitive

apiVersion: v1
kind: Pod
metadata:
	name: nginx
	labels:
		app: nginx
		tier: frontend
spec:
	containers:
		- name: nginx
		  image: nginx

--------------------------------------------------------------------------------------------------

kubectl apply -f pod.yml	




kubectl get nodes -> to check nodes running
kubectl describe nodes - > 	to get version of OS where nodes is running

-------starting pod---------------
root@controlplane:~# kubectl run nginx --image=nginx
pod/nginx created
root@controlplane:~# kubectl get pods
NAME    READY   STATUS              RESTARTS   AGE
nginx   0/1     ContainerCreating   0          7s
root@controlplane:~# 

-------What is the image used in the pod--------
root@controlplane:~# kubectl get pods
NAME            READY   STATUS    RESTARTS   AGE
newpods-42rq5   1/1     Running   0          88s
newpods-gjwj7   1/1     Running   0          88s
newpods-qn46q   1/1     Running   0          88s
nginx           1/1     Running   0          2m38s

kubectl describe pods newpods-gjwj7

--------Delete the existing Pod ---------------
oot@controlplane:~# kubectl get pods
NAME            READY   STATUS             RESTARTS   AGE
newpods-42rq5   1/1     Running            0          11m
newpods-gjwj7   1/1     Running            0          11m
newpods-qn46q   1/1     Running            0          11m
nginx           1/1     Running            0          12m
webapp          1/2     ImagePullBackOff   0          4m32s
root@controlplane:~# kubectl delete webapp
error: the server doesn't have a resource type "webapp"
root@controlplane:~# kubectl delete pod webapp
pod "webapp" deleted

--------------create POD with YML------------------------------------
root@controlplane:~# kubectl apply -f pod1.yml 
pod/redis created
root@controlplane:~# kubectl get pods
NAME            READY   STATUS             RESTARTS   AGE
newpods-42rq5   1/1     Running            0          14m
newpods-gjwj7   1/1     Running            0          14m
newpods-qn46q   1/1     Running            0          14m
nginx           1/1     Running            0          15m
redis           0/1     ImagePullBackOff   0          10s
root@controlplane:~# kubectl describe pod redis

--------update the yml file and rerun the pod-----------
> update image name correctly in yml file.
kubectl edit pod redis

======================================================================================================
Replication Controllers and ReplicaSets
======================================================================================================

---run the replicationcontroller definition file----
kubectl create -f rc.yml

---check status of the replication controller ---
kubectl get replicationcontroller

---check the replicasets status---
kubectl get replicasets
NAME              DESIRED   CURRENT   READY   AGE
new-replica-set   4         4         0       6s
root@controlplane:~# 

---create replicaset ---
kubectl create  -f /root/replicaset-definition-1.yaml 
replicaset.apps/replicaset-1 created

root@controlplane:~# kubectl get replicaset 
NAME              DESIRED   CURRENT   READY   AGE
new-replica-set   4         4         0       9m46s
replicaset-1      2         2         0       15s

root@controlplane:~# kubectl get pods
NAME                    READY   STATUS             RESTARTS   AGE
replicaset-1-c87tf      1/1     Running            0          34s
replicaset-1-m8b7z      1/1     Running            0          34s

----------if in case, you dont have yml to edit the replicaset , then how to modify running repicaset yml--- 
kubectl edit replicaset myapp-replicaset
-> post above delete the pods , it will create new pods with updated replicaset yml

----scale up/down the pod count ---
kubectl scale --replicas=5 replicaset new-replica-set

---> the appversion for - rc is v1 & replicaset its - apps/v1


============================ deployment =================================

kubectl get all -> to get all objects information in single shot instead of seperate command for each - pod, deployment, replicaset

kubectl deployment -f deployment-definition-2.yaml
 kubectl delete deployment.apps/deployment12


root@controlplane:~# kubectl get all
NAME                                       READY   STATUS             RESTARTS   AGE
pod/httpd-frontend-7df5cf55f-6kfx6         1/1     Running            0          72s
pod/httpd-frontend-7df5cf55f-d7mrp         1/1     Running            0          72s
pod/httpd-frontend-7df5cf55f-fk79r         1/1     Running            0          72s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   55m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/httpd-frontend        3/3     3            3           73s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/httpd-frontend-7df5cf55f         3         3         3       72s
root@controlplane:~# 



All thee 1, 2 , 3 should match
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-frontend  -------------------1
spec:
  replicas: 3
  selector:
    matchLabels:
      name: httpd-frontend ---------------- 2
  template:
    metadata:
      labels:
        name: httpd-frontend ---------------- 3
    spec:
      containers:
      - name: httpd-frontend
        image: httpd:2.4-alpine

------- rollout and versioning-----------------------

deployment strategies:
1. recreate strategy : destroy all instances & then create all instance - recreate strategy - downtime expected. it is not default
2. rolling strategy: this will destroy one instance and create , then move onto next instance one at a time.


kind: Recreate ; kind: RollingUpdate

commands need to capture from -> https://www.udemy.com/course/learn-kubernetes/learn/lecture/9723324#overview

=====================================================================================================================================================================

Things to explore container:  - app and type attributes.

================================================Servcices===========================================================================

root@controlplane:~# cat service-definition-1.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: NodePort
  ports:
    - targetPort: 8080
      port: 8080
      nodePort: 30080
  selector:
    name: simple-webapp
root@controlplane:~# 


