Creating cluster by using config.yaml

- kind create cluster --name surya --config config.yaml

# #########################################################

In Kubernetes, Requests and Limits are resource configurations that help manage how much CPU and memory a container is allocated within a cluster. They ensure fair resource sharing and prevent containers from overloading the system.

# Resource Requests
Definition: The amount of CPU or memory guaranteed to a container.

Behavior:

Kubernetes schedules the container on a node that has enough capacity to meet its request.
If a container’s workload increases, it can exceed its request (up to the limit, if set).
Example Use Case: Ensures your application has a minimum amount of resources to function effectively.
Units:
CPU: Fraction of a core (0.5 = 500 millicores).
Memory: Specified in bytes, KiB, MiB, or GiB (256Mi = 256 Megabytes).

# Resource Limits
Definition: The maximum amount of CPU or memory a container can use.

Behavior:
If a container exceeds its memory limit, it is terminated with an *OOMKilled (Out of Memory)* error.
If it exceeds its CPU limit, Kubernetes throttles the CPU (slows down processing).
Example Use Case: Prevents a single container from consuming all resources and affecting other applications.

Why Use Requests and Limits?

Stability: Prevents noisy neighbor problems (one container using all resources at the expense of others).
Fair Resource Sharing: Ensures every application gets its fair share of resources.

Cost Efficiency: Helps optimize resource allocation, avoiding over-provisioning or under-provisioning.

-----------------------------------------------------------
Example YAML Configuration:


apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "256Mi"
        cpu: "500m"
      limits:
        memory: "512Mi"
        cpu: "1000m"

Requests:
Guaranteed minimum: 256Mi memory, 500m (0.5 core) CPU.
Limits:
Maximum: 512Mi memory, 1000m (1 core) CPU.


# Key Concepts

Guaranteed: If requests = limits, the container gets dedicated resources and won’t be throttled.
Burstable: If requests < limits, the container can burst resource usage temporarily but may be throttled.
Best-Effort: If no requests or limits are set, the container competes for resources with the lowest priority.
Best Practices
Set Requests and Limits: Always define both to prevent over-commitment or under-utilization.
Monitor Resource Usage: Use tools like Prometheus, Grafana, or Kubernetes Metrics Server to analyze actual usage.
Test and Adjust: Start with conservative estimates and adjust based on your application’s behavior.

- monitoring these metrics or fine-tuning them

# Installing Metrics-server

 kubectl apply -f .\metrics-server.yaml

serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created


 kubectl get po -n kube-system
NAME  READY   STATUS    RESTARTS   AGE
coredns-7c65d6cfc9-28s7h  1/1     Running   025m
coredns-7c65d6cfc9-pngl5  1/1     Running   025m
etcd-surya-control-plane  1/1     Running   025m
kindnet-nc55r   1/1     Running   025m
kindnet-ps9pb   1/1     Running   025m
kindnet-zqqcc   1/1     Running   025m
kube-apiserver-surya-control-plane  1/1     Running   025m
kube-controller-manager-surya-control-plane   1/1     Running   025m
kube-proxy-8rjjn1/1     Running   025m
kube-proxy-gtzzn1/1     Running   025m
kube-proxy-hkdcj1/1     Running   025m
kube-scheduler-surya-control-plane  1/1     Running   025m
metrics-server-699f9ddd54-xgmsn     1/1     Running   028s


kubectl top nodes

NAME        CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
surya-control-plane   74m0%     844Mi 22%
surya-worker14m0%     271Mi 7%
surya-worker2         12m0%     220Mi 5%



 kubectl top nodes
 
NAME        CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
surya-control-plane   74m0%     844Mi 22%
surya-worker14m0%     271Mi 7%
surya-worker2         12m0%     220Mi 5%

# Created Namespace

namespace/mem-exampls created


 kubectl apply -f .\requests.yaml
pod/memory-demo created

 kubectl get pods -n mem-exampls
NAMEREADY   STATUS    RESTARTS   AGE
memory-demo   1/1     Running   06s

 kubectl top nodes
NAME        CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
surya-control-plane   75m0%     857Mi 22%
surya-worker15m0%     273Mi 7%
surya-worker2         12m0%     276Mi 7%



kubectl top nodes
NAME        CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
surya-control-plane   70m0%     861Mi 22%       
surya-worker13m0%     273Mi 7%        
surya-worker2         13m0%     279Mi 7%        


 kubectl get nodes -o wide

NAME        STATUS   ROLES AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE     KERNEL-VERSION   CONTAINER-RUNTIME
surya-control-plane   Ready    control-plane   58m   v1.31.2   172.18.0.4    <none>        Debian GNU/Linux 12 (bookworm)   5.15.167.4-microsoft-standard-WSL2   containerd://1.7.18
surya-workerReady    <none>58m   v1.31.2   172.18.0.2    <none>        Debian GNU/Linux 12 (bookworm)   5.15.167.4-microsoft-standard-WSL2   containerd://1.7.18
surya-worker2         Ready    <none>58m   v1.31.2   172.18.0.3    <none>        Debian GNU/Linux 12 (bookworm)   5.15.167.4-microsoft-standard-WSL2   containerd://1.7.18

 kubectl get nodes -o custom-columns=NAME:.metadata.name,CPU:.status.capacity.cpu,MEMORY:.status.capacity.memory

NAME        CPU   MEMORY
surya-control-plane   8     3901464Ki
surya-worker8     3901464Ki
surya-worker2         8     3901464Ki


 kubectl apply -f .\stress.yaml
pod/memory-demo-2 created

 kubectl topnodes         
NAME        CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
surya-control-plane   83m1%     862Mi 22%
surya-worker14m0%     273Mi 7%
surya-worker2         46m0%     286Mi 7%
 kubectl get pods -n mem-exampls     
NAME  READY   STATUS   RESTARTS     AGE
memory-demo     1/1     Running  0  18m
memory-demo-2   0/1     CrashLoopBackOff   1 (2s ago)   19s

 kubectl describe po/memory-demo-2 -n mem-exampls 


Name:   memory-demo-2
Namespace:        mem-exampls
Priority:         0
Service Account:  default
Node:   surya-worker2/172.18.0.3
Start Time:       Wed, 11 Dec 2024 20:58:54 +0530
Labels: <none>
Annotations:      <none>
Status: Running
IP:     10.244.2.4
IPs:
  IP:  10.244.2.4
Containers:
  memory-demo:
    Container ID:  containerd://bda9bbd26753ee08ad1ac8dd8927136235d8c5cfeabbdfa48fb89e4dd623addb
    Image:         polinux/stress
    Image ID:      docker.io/polinux/stress@sha256:b6144f84f9c15dac80deb48d3a646b55c7043ab1d83ea0a697c09097aaad21aa
    Port:<none>
    Host Port:     <none>
    Command:
      stress
    Args:
      --vm
      1
      --vm-bytes
      250M
      --vm-hang
      1
    State:Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
      Started:      Wed, 11 Dec 2024 20:59:31 +0530
      Finished:     Wed, 11 Dec 2024 20:59:32 +0530
    Ready:False
    Restart Count:  2
    Limits:
      memory:  100Mi
    Requests:
      memory:     50Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-k5qnh (ro)
Conditions:
  Type    Status
  PodReadyToStartContainers   True
  Initialized       True
  Ready   False
  ContainersReady   False
  PodScheduled      True
Volumes:
  kube-api-access-k5qnh:
    Type:Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName: kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:   true
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
         node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age      From     Message
  ----     ------     ----     ----     -------
  Normal   Scheduled  57s      default-scheduler  Successfully assigned mem-exampls/memory-demo-2 to surya-worker2
  Normal   Pulled     44s      kubelet  Successfully pulled image "polinux/stress" in 12.472s (12.472s including waiting). Image size: 4041495 bytes.
  Normal   Pulled     40s      kubelet  Successfully pulled image "polinux/stress" in 1.892s (1.892s including waiting). Image size: 4041495 bytes.
  Normal   Pulling    25s (x3 over 56s)  kubelet  Pulling image "polinux/stress"
  Normal   Created    20s (x3 over 44s)  kubelet  Created container memory-demo
  Normal   Started    20s (x3 over 44s)  kubelet  Started container memory-demo
  Normal   Pulled     20s      kubelet  Successfully pulled image "polinux/stress" in 5.655s (5.655s including waiting). Image size: 4041495 bytes.
  Warning  BackOff    6s (x4 over 39s)   kubelet  Back-off restarting failed container memory-demo in pod memory-demo-2_mem-exampls(3a514cfa-76b3-4b0e-a9b0-5de710efb0ae)


 kubectl get pods -n mem-exampls


NAME  READY   STATUS    RESTARTS      AGE
memory-demo     1/1     Running   0   19m
memory-demo-2   0/1     Error     3 (36s ago)   74s

 kubectl get pods -n mem-exampls

NAME  READY   STATUS   RESTARTS      AGE
memory-demo     1/1     Running  0   20m
memory-demo-2   0/1     CrashLoopBackOff   4 (31s ago)   2m22s

 kubectl get pod memory-demo-2 -n mem-exampls -o yaml


apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"memory-demo-2","namespace":"mem-exampls"},"spec":{"containers":[{"args":["--vm","1","--vm-bytes","250M","--vm-hang","1"],"command":["stress"],"image":"polinux/stress","name":"memory-demo","resources":{"limits":{"memory":"100Mi"},"requests":{"memory":"50Mi"}}}]}}
  creationTimestamp: "2024-12-11T15:28:54Z"
  name: memory-demo-2
  namespace: mem-exampls
  resourceVersion: "6911"
  uid: 3a514cfa-76b3-4b0e-a9b0-5de710efb0ae
spec:
  containers:
  - args:
    - --vm
    - "1"
    - --vm-bytes
    - 250M
    - --vm-hang
    - "1"
    command:
    - stress
    image: polinux/stress
    imagePullPolicy: Always
    name: memory-demo
    resources:
      limits:
        memory: 100Mi
      requests:
        memory: 50Mi
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-k5qnh
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: surya-worker2
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-k5qnh
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
expirationSeconds: 3607
path: token
      - configMap:
items:
- key: ca.crt
  path: ca.crt
name: kube-root-ca.crt
      - downwardAPI:
items:
- fieldRef:
    apiVersion: v1
    fieldPath: metadata.namespace
  path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2024-12-11T15:29:08Z"
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2024-12-11T15:28:54Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2024-12-11T15:30:02Z"
    message: 'containers with unready status: [memory-demo]'
    reason: ContainersNotReady
    status: "False"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2024-12-11T15:30:02Z"
    message: 'containers with unready status: [memory-demo]'
    reason: ContainersNotReady
    status: "False"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2024-12-11T15:28:54Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://b60755e142288c9c52dd38d175b3729aa2dab2f01cb84eaced8068d89622793b
    image: docker.io/polinux/stress:latest
    imageID: docker.io/polinux/stress@sha256:b6144f84f9c15dac80deb48d3a646b55c7043ab1d83ea0a697c09097aaad21aa
    lastState:
      terminated:
        containerID: containerd://b60755e142288c9c52dd38d175b3729aa2dab2f01cb84eaced8068d89622793b
        exitCode: 1
        finishedAt: "2024-12-11T15:35:04Z"
        reason: Error
        startedAt: "2024-12-11T15:35:04Z"
    name: memory-demo
    ready: false
    restartCount: 6
    started: false
    state:
      waiting:
        message: back-off 5m0s restarting failed container=memory-demo pod=memory-demo-2_mem-exampls(3a514cfa-76b3-4b0e-a9b0-5de710efb0ae)
        reason: CrashLoopBackOff
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-k5qnh
      readOnly: true
      recursiveReadOnly: Disabled
  hostIP: 172.18.0.3
  hostIPs:
  - ip: 172.18.0.3
  phase: Running
  podIP: 10.244.2.4
  podIPs:
  - ip: 10.244.2.4
  qosClass: Burstable
  startTime: "2024-12-11T15:28:54Z"
 kubectl get pods -a
error: unknown shorthand flag: 'a' in -a
See 'kubectl get --help' for usage

 kubectl get pods -A


NAMESPACE  NAME  READY   STATUS   RESTARTS      AGE
kube-systemcoredns-7c65d6cfc9-28s7h  1/1     Running  0   69m
kube-systemcoredns-7c65d6cfc9-pngl5  1/1     Running  0   69m
kube-systemetcd-surya-control-plane  1/1     Running  0   69m
kube-systemkindnet-nc55r   1/1     Running  0   69m
kube-systemkindnet-ps9pb   1/1     Running  0   69m
kube-systemkindnet-zqqcc   1/1     Running  0   69m
kube-systemkube-apiserver-surya-control-plane  1/1     Running  0   70m
kube-systemkube-controller-manager-surya-control-plane   1/1     Running  0   69m
kube-systemkube-proxy-8rjjn1/1     Running  0   69m
kube-systemkube-proxy-gtzzn1/1     Running  0   69m
kube-systemkube-proxy-hkdcj1/1     Running  0   69m
kube-systemkube-scheduler-surya-control-plane  1/1     Running  0   69m
kube-systemmetrics-server-699f9ddd54-xgmsn     1/1     Running  0   44m
local-path-storage   local-path-provisioner-57c5987fd4-crgdr       1/1     Running  0   69m
mem-examplsmemory-demo     1/1     Running  0   25m
mem-examplsmemory-demo-2   0/1     CrashLoopBackOff   6 (69s ago)   7m19s

# - - -------------------

kubectl apply -f .\stress2.yaml

pod/memory-demo-2 created

 kubectl get pods -n mem-exampls
NAME  READY   STATUS    RESTARTS   AGE
memory-demo     1/1     Running   037m
memory-demo-2   1/1     Running   017s


 kubectl get pods -n mem-exampls
NAME  READY   STATUS    RESTARTS   AGE
memory-demo     1/1     Running   037m
memory-demo-2   1/1     Running   049s



 kubectl get deployment metrics-server -n kube-system
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
metrics-server   1/1     1  1 58m


 kubectl logs -n kube-system deployment/metrics-server


 kubectl top pod memory-demo-2 -n mem-exampls


NAME  CPU(cores)   MEMORY(bytes)   
memory-demo-2   16m150Mi


 kubectl top nodes

NAME        CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
surya-control-plane   81m1%     878Mi 23%
surya-worker16m0%     268Mi 7%
surya-worker2         27m0%     444Mi 11%


 kubectl get pods -o wide -A

NAMESPACE  NAME  READY   STATUS    RESTARTS   AGE    IP NODE        NOMINATED NODE   READINESS GATES
kube-systemcoredns-7c65d6cfc9-28s7h  1/1     Running   085m    10.244.0.4   surya-control-plane   <none>
 <none>
kube-systemcoredns-7c65d6cfc9-pngl5  1/1     Running   085m    10.244.0.2   surya-control-plane   <none>
 <none>
kube-systemetcd-surya-control-plane  1/1     Running   085m    172.18.0.4   surya-control-plane   <none>
 <none>
kube-systemkindnet-nc55r   1/1     Running   085m    172.18.0.2   surya-worker<none>
 <none>
kube-systemkindnet-ps9pb   1/1     Running   085m    172.18.0.3   surya-worker2         <none>
 <none>
kube-systemkindnet-zqqcc   1/1     Running   085m    172.18.0.4   surya-control-plane   <none>
 <none>
kube-systemkube-apiserver-surya-control-plane  1/1     Running   085m    172.18.0.4   surya-control-plane   <none>
 <none>
kube-systemkube-controller-manager-surya-control-plane   1/1     Running   085m    172.18.0.4   surya-control-plane   <none>
 <none>
kube-systemkube-proxy-8rjjn1/1     Running   085m    172.18.0.4   surya-control-plane   <none>
 <none>
kube-systemkube-proxy-gtzzn1/1     Running   085m    172.18.0.3   surya-worker2         <none>
 <none>
kube-systemkube-proxy-hkdcj1/1     Running   085m    172.18.0.2   surya-worker<none>
 <none>
kube-systemkube-scheduler-surya-control-plane  1/1     Running   085m    172.18.0.4   surya-control-plane   <none>
 <none>
kube-systemmetrics-server-699f9ddd54-xgmsn     1/1     Running   060m    10.244.1.2   surya-worker<none>
 <none>
local-path-storage   local-path-provisioner-57c5987fd4-crgdr       1/1     Running   085m    10.244.0.3   surya-control-plane   <none>
 <none>
mem-examplsmemory-demo     1/1     Running   041m    10.244.2.3   surya-worker2         <none>
 <none>
mem-examplsmemory-demo-2   1/1     Running   04m2s   10.244.2.6   surya-worker2         <none>

# Please folow Below Commands 
 
   - kind create cluster --name surya --config .\config.yaml
   - kubectl top node
   - kubectl get nodes
   - docker ps -a
   - docker exec -it containerid bash
   - kubectl get pod
   - kubectl get pod -n kube-system
   - kubectl apply -f .\metrics-server.yaml
   - kubectl get po
   - kubectl get po -n kube-system
   - kubectl describe po/metrics-server-699f9ddd54-xgmsn  -n kube-system
   - kubectl get po -n kube-system
   - kubectl get nodes -o wide
   - kubectl describe po/memory-demo-2 -n mem-exampls
   - kubectl get pod memory-demo-2 -n mem-exampls -o yaml
  - kubectl get pods -A
  - kubectl logs -n kube-system deployment/metrics-server
  - kubectl describe pod memory-demo-2 -n mem-exampls
  - kubectl top pod memory-demo-2 -n mem-exampls