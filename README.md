# Taint-Toleration-Node-Affinity

- Step 1: Firt identify your nodes
``` bash
kubectl get nodes
```
OUTPUT:
``` bash
NAME                 STATUS   ROLES           AGE   VERSION
prod-control-plane   Ready    control-plane   18h   v1.29.2
prod-worker          Ready    <none>          18h   v1.29.2
prod-worker2         Ready    <none>          18h   v1.29.2
```
You add a taint to a node using kubectl taint. For example
- kubectl taint nodes node1 key1=value1:<taint-effect>
-There are 3 effect

NoSchedule : No new Pods will be scheduled on the tainted node unless they have a matching toleration. Pods currently running on the node are not evicted.
preferNoSchedule : soft version of NoSchedule. The control plane will try to avoid placing a Pod that does not tolerate the taint on the node, but it is not guaranteed.
NoExecute: Pods that do not tolerate the taint are evicted immediately

_ Taint the Node for Frontend Environment 

``` bash
kubectl taint nodes prod-worker app=frontend:NoSchedule
```
_ Taint the Node for Data Base Environment 

``` bash
kubectl taint nodes prod-worker2 app=database:NoSchedule
```
- To confirm the taint effect on Nodes
``` bash
kubectl describe node prod-worker | grep Taints

kubectl describe node prod-worker2 | grep Taints
```

- To remove the taint added by the command above, you can run:
``` bash
kubectl taint nodes node1 app=frontend:NoSchedule-
```
- Step 2: Specify a toleration for a pod in configuration file
- FrontEnd Configuration Yaml File
``` bash
apiVersion: v1
kind: Pod
metadata:
  name: frontend-nginx-pod
  labels:
    env: test
spec:
  containers:
  - name: nginx-container
    image: nginx
  tolerations:
  - key: "app"
    value: "frontend"
    effect: "NoSchedule"
```
- Deploy the FrontEnd Pod
``` bash
kubectl apply -f frontend-pod.yaml
``` 
- Back End Configuration Yaml File
``` bash
apiVersion: v1
kind: Pod
metadata:
  name: database-mysql-pod
  labels:
    env: test
spec:
  containers:
  - name: database-container
    image: mysql
  tolerations:
  - key: "app"
    value: "databas"
    effect: "NoSchedule"
```
- Deploy the BackEnd Pod
``` bash
kubectl apply -f backend-pod.yaml
```
- Step 3: Label the Node to apply the Node Affinity
``` bash
kubectl label node prod-worker app=user
```
- To check the all labels on a specfic node
``` bash
 kubectl describe node prod-worker | grep -i Labels -5
```
- Step 4: Add the node Affinity in configuration file

Note:there are 3 different types of node affinity effects
- requiredDuringSchedulingIgnoredDuringExecution
- preferredDuringSchedulingIgnoredDuringExecution
- requiredDuringSchedulingrequiredDuringExecution
there are 3 different types of operator in node affinity 
- NotIn
- In
- Exists

``` bash
 apiVersion: v1
kind: Pod
metadata:
  name: frontend-nginx-pod
  labels:
    env: test
spec:
  containers:
  - name: nginx-container
    image: nginx
  tolerations:
  - key: "app"
    value: "frontend"
    effect: "NoSchedule"
affinity:
  nodeAffinity:
    requiredDuringSchedulingRequiredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: app
          operator: In
          values:
          - user
```
