Node Selectors are great for basic pod placement based on node labels. But what if you need more control over where your pods land? Enter Node Affinity! This feature offers advanced capabilities to fine-tune pod scheduling in your Kubernetes cluster.

# Node Affinity:

Node Affinity lets you define complex rules for where your pods can be scheduled based on node labels. Think of it as creating a wishlist for your pod's ideal home!

# Key Features:

Flexibility: Define precise conditions for pod placement.
Control: Decide where your pods can and cannot go with greater granularity.
Adaptability: Allow pods to stay on their nodes even if the labels change after scheduling.

Properties in Node Affinity

requiredDuringSchedulingIgnoredDuringExecution
preferredDuringSchedulingIgnoredDuringExecution
Required During Scheduling, Ignored During Execution 

This is the strictest type of Node Affinity. Here's how it works:

Specify Node Labels: Define a list of required node labels (e.g., disktype=ssd) in your pod spec.
Exact Match Requirement: The scheduler only places the pod on nodes with those exact labels.
Execution Consistency: Once scheduled, the pod remains on the node even if the label changes.



create a pod with nginx as the image and add the nodeffinity with property requiredDuringSchedulingIgnoredDuringExecution and condition disktype = ssd

# requiredDuringSchedulingIgnoredDuringExecution:

refer : nodeaffinity.yaml

kubectl create ns surya
namespace/surya created
\kubernetes\NodeAffinity> kubectl apply -f .\nodeaffinity.yaml
pod/my-affinity created

kubectl get pods -n surya      
NAME          READY   STATUS    RESTARTS   AGE
my-affinity   0/1     Pending   0          39s

Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  44s   default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }, 2 node(s) didn't match Pod's node affinity/selector. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.

# preferredDuringSchedulingIgnoredDuringExecution:

 kubectl apply -f .\nodeaffinity2.yaml
pod/my-affinitynew created
\kubernetes\NodeAffinity> kubectl get pods -n surya
NAME             READY   STATUS    RESTARTS   AGE
my-affinity      0/1     Pending   0          8m34s
my-affinitynew   1/1     Running   0          17s
\kubernetes\NodeAffinity>

# kubectl get pods -o wide -n surya
NAME             READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
my-affinity      0/1     Pending   0          11m     <none>       <none>         <none>           <none>
my-affinitynew   1/1     Running   0          3m13s   10.244.2.2   kind-worker2   <none>           <none>

\kubernetes\NodeAffinity> kubectl get nodes
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   37m   v1.31.2
kind-worker          Ready    <none>          37m   v1.31.2
kind-worker2         Ready    <none>          37m   v1.31.2

# kubernetes\NodeAffinity> kubectl label node kind-worker disktype=ssd
node/kind-worker labeled

\kubernetes\NodeAffinity> kubectl get pods -o wide -n surya
NAME             READY   STATUS              RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
my-affinity      0/1     ContainerCreating   0          12m     <none>       kind-worker    <none>           <none>
my-affinitynew   1/1     Running             0          4m29s   10.244.2.2   kind-worker2   <none>           <none>


\kubernetes\NodeAffinity> kubectl get pods -o wide -n surya
NAME             READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
my-affinity      1/1     Running   0          12m     10.244.1.2   kind-worker    <none>           <none>
my-affinitynew   1/1     Running   0          4m42s   10.244.2.2   kind-worker2   <none>           <none>
\kubernetes\NodeAffinity>


# no Matched key label


kubectl get pods -o wide -n surya            
NAME             READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
my-affinity      1/1     Running   0          32m     10.244.1.2   kind-worker    <none>           <none>
my-affinitynew   1/1     Running   0          24m     10.244.2.2   kind-worker2   <none>           <none>
my-noaffinity    0/1     Pending   0          3m11s   <none>       <none>         <none>           <none>


kubernetes\NodeAffinity> kubectl get node --show-labels
NAME                 STATUS   ROLES           AGE   VERSION   LABELS
kind-control-plane   Ready    control-plane   60m   v1.31.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kind-control-plane,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=   
kind-worker          Ready    <none>          60m   v1.31.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=ssd,kubernetes.io/arch=amd64,kubernetes.io/hostname=kind-worker,kubernetes.io/os=linux
kind-worker2         Ready    <none>          60m   v1.31.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=,kubernetes.io/arch=amd64,kubernetes.io/hostname=kind-worker2,kubernetes.io/os=linux

kubernetes\NodeAffinity> kubectl get pods -o wide -n surya

NAME             READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
my-affinity      1/1     Running   0          36m     10.244.1.2   kind-worker    <none>           <none>
my-affinitynew   1/1     Running   0          28m     10.244.2.2   kind-worker2   <none>           <none>
my-noaffinity    0/1     Pending   0          6m43s   <none>       <none>         <none>           <none>

kubernetes\NodeAffinity> kubectl delete po/my-noaffinity -n surya
pod "my-noaffinity" deleted

# kubectl label node kind-worker2 disktype=

kubernetes\NodeAffinity> kubectl apply -f .\noaffinity.yaml 
pod/my-noaffinity created

kubernetes\NodeAffinity> kubectl get pods -o wide -n surya  
    
NAME             READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
my-affinity      1/1     Running   0          37m   10.244.1.2   kind-worker    <none>           <none>
my-affinitynew   1/1     Running   0          28m   10.244.2.2   kind-worker2   <none>           <none>
my-noaffinity    1/1     Running   0          5s    10.244.1.3   kind-worker    <none>           <none>

below commands are usefull:

    - kubectl get po -o wide -n surya
    - kubectl label node node-name disktype=ssd
    - kubectl apply -f yaml_name
    - kubectl get node --show-labels 
    - kubectl label node kind-worker2 disktype -
    - kubectl describe po/my-noaffinity -n surya

------------------------------------------------------

  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd

-------------------------------------------------------

  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference:
            matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd 
                  
--------------------------------------------------------
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: "disktype"
                operator: "Exists"
--------------------------------------------------------


