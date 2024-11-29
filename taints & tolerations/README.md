
# TAINTS

Definition: A taint is applied to a Node to repel Pods from being scheduled on it unless they have a matching toleration



# Tolerations

Definition: Tolerations are applied to Pods to allow them to be scheduled onto Nodes with matching taints.

# Taint Effects
NoSchedule: Pods without a matching toleration will not be scheduled on the Node.
PreferNoSchedule: Kubernetes will try to avoid scheduling Pods without a matching toleration but won't enforce it strictly.
NoExecute: Pods without a matching toleration will be evicted from the Node if they are already running and will not be scheduled in the future.
# Toleration Fields

key: Matches the taint key.
operator:
Equal: Matches if the key and value are equal.
Exists: Matches only the key (value is not required).
value: Matches the taint value (used with Equal operator).
effect: Matches the taint effect (NoSchedule, PreferNoSchedule, NoExecute).

## IMP ##

Dedicated Nodes: Prevent non-critical workloads from running on specific Nodes.
Node Maintenance: Use NoExecute taints to evict Pods from a Node temporarily.
Resource Specialization: Schedule specialized workloads (e.g., GPU, high memory) on appropriate Nodes.


worker01--> gpu=true:NoSchedule , worker02--> gpu=false:NoSchedule


# kubectl describe node surya-worker       
Name:               surya-worker
Roles:              <none>
CreationTimestamp:  Fri, 29 Nov 2024 13:51:59 +0530
Taints:             gpu=true:NoSchedule

# kubectl describe node surya-worker2
Name:               surya-worker2
Roles:              <none>
CreationTimestamp:  Fri, 29 Nov 2024 13:51:59 +0530
Taints:             gpu=false:NoSchedule

# kubectl apply -f .\pod.yaml
pod/surya-app created
kubernetes\taints & tolerations> kubectl get pod -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
surya-app   0/1     Pending   0          14s   <none>   <none>   <none>           <none>
kubernetes\taints & tolerations> kubectl describe po/surya-app

`POD not created Becuse there no matching tolarations - taints : controal-palne has bydeafult taint, and workers we have attahed taints `

Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  30s   default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint {gpu: false}, 1 node(s) had untolerated taint {gpu: true}, 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.     

 kubectl get pod -o wide
NAME        READY   STATUS    RESTARTS   AGE    IP       NODE     NOMINATED NODE   READINESS GATES
surya-app   0/1     Pending   0          116s   <none>   <none>   <none>           <none>
kubernetes\taints & tolerations> kubectl get pod -o wide
NAME        READY   STATUS    RESTARTS   AGE    IP       NODE     NOMINATED NODE   READINESS GATES
surya-app   0/1     Pending   0          2m4s   <none>   <none>   <none>           <none>
kubernetes\taints & tolerations> kubectl get pod -o wide


Create a new pod with the image nginx and see why it's not getting scheduled on worker nodes and control plane nodes.

# Create a toleration on the pod gpu=true:NoSchedule to match with the taint on worker01

refer : toleration-pod.yaml value is not provides getting bellow error.


Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  51s   default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint {gpu: false}, 1 node(s) had untolerated taint {gpu: true}, 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.   


# kubectl apply -f .\toleration-pod.yaml

pod/surya-app created
kubernetes\taints & tolerations> kubectl get pods -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
surya-app   0/1     Pending   0          13s   <none>   <none>   <none>           <none>
kubernetes\taints & tolerations> kubectl get pods -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
surya-app   0/1     Pending   0          20s   <none>   <none>   <none>           <none>
kubernetes\taints & tolerations> kubectl desrcibe po/surya-app

`kubectl taint node surya-worker gpu=true:NoSchedule `  

kubectl apply -f .\toleration-pod.yaml
pod/surya-app created
   kubernetes\taints & tolerations> kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
surya-app   1/1     Running   0          7s


Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  21s   default-scheduler  Successfully assigned default/surya-app to `surya-worker`
  Normal  Created    19s   kubelet            Created container my-surya-app
  Normal  Started    19s   kubelet            Started container my-surya-ap

`kubectl taint node surya-worker2 gpu=false:NoSchedule `    

kubectl apply -f .\toleration-pod1.yaml
pod/surya-appnew created
kubernetes\taints & tolerations> kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
surya-app      1/1     Running   0          5m45s
surya-appnew   1/1     Running   0          11s



Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  29s   default-scheduler  Successfully assigned default/surya-appnew to surya-worker2
  Normal  Created    27s   kubelet            Created container my-surya-appnew
  Normal  Started    27s   kubelet            Started container my-surya-appnew

`Now pod creating by deleting taints`

# kubectl taints node surya-worker gpu=true:NoSchedule -

kubectl apply -f .\pod.yaml
pod/surya-app created

kubectl get po
NAME        READY   STATUS    RESTARTS   AGE
surya-app   0/1     Pending   0          22s

kubectl taint nodes surya-worker gpu=true:NoSchedule-
node/surya-worker untainted
PS C:\Users\karri\OneDrive\Desktop\kind\kubernetes\taints & tolerations> kubectl get po                                       
NAME        READY   STATUS    RESTARTS   AGE
surya-app   1/1     Running   0          4m51s
PS C:\Users\karri\OneDrive\Desktop\kind\kubernetes\taints & tolerations>