## Labels and Selectors in Kubernetes

### Labels 
Labels are key-value pairs attached to Kubernetes objects like pods, services, and deployments. They help organize and group resources based on criteria that make sense to you.

**Examples of Labels:**
- `environment: production`
- `type: backend`
- `tier: frontend`
- `application: my-app`

### Selectors
Selectors filter Kubernetes objects based on their labels. This is incredibly useful for querying and managing a subset of objects that meet specific criteria.

**Common Usage:**
- **Pods**: `kubectl get pods --selector app=my-app`
- **Deployments**: Used to filter the pods managed by the deployment.
- **Services**: Filter the pods to which the service routes traffic.

### Labels vs. Namespaces 
- **Labels**: Organize resources within the same or across namespaces.
- **Namespaces**: Provide a way to isolate resources from each other within a cluster.

### Annotations
Annotations are similar to labels but attach non-identifying metadata to objects. For example, recording the release version of an application for information purposes or last applied configuration details etc.

---

## Static Pods

Static Pods are special types of pods managed directly by the `kubelet` on each node rather than through the Kubernetes API server.

### Key Characteristics of Static Pods:
- **Not Managed by the Scheduler**: Unlike deployments or replicasets, the Kubernetes scheduler does not manage static pods.
- **Defined on the Node**: Configuration files for static pods are placed directly on the node's file system, and the `kubelet` watches these files.
- **Some examples of static pods are:** ApiServer, Kube-scheduler, controller-manager, ETCD etc
  
### Managing Static Pods:
1. **SSH into the Node**: You will gain access to the node where the static pod is defined.(Mostly the control plane node)
2. **Modify the YAML File**: Edit or create the YAML configuration file for the static pod.
3. **Remove the Scheduler YAML**: To stop the pod, you must remove or modify the corresponding file directly on the node.
4. **Default location**": is usually `/etc/kubernetes/manifests/`; you can place the pod YAML in the directory, and Kubelet will pick it for scheduling.

## Manual Pod Scheduling

Manual scheduling in Kubernetes involves assigning a pod to a specific node rather than letting the scheduler decide.

### Key Points:
- **`nodeName` Field**: Use this field in the pod specification to specify the node where the pod should run.
- **No Scheduler Involvement**: When `nodeName` is specified, the scheduler bypasses the pod, and it’s directly assigned to the given node.

# refer :  manual-schedule-pod.yaml

 kubectl apply -f .\manual-schedule-pod.yaml
pod/my-nodename created

`Now Schedular scheduled the pod in worker node 2`
 kubectl get pods -o wide
NAME          READY   STATUS    RESTARTS   AGE   IP           NODE            NOMINATED NODE   READINESS GATES
my-nodename   1/1     Running   0          79s   10.244.1.2   surya-worker2   <none>           <none>

I have removed kube-scheduler.yaml in manifest, now apply manual-schedule-pod1.yaml

root@surya-control-plane:/etc/kubernetes/manifests# pwd
/etc/kubernetes/manifests
root@surya-control-plane:/etc/kubernetes/manifests# ls 
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
root@surya-control-plane:/etc/kubernetes/manifests# mv kube-scheduler.yaml /etc/
root@surya-control-plane:/etc/kubernetes/manifests# ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml
root@surya-control-plane:/etc/kubernetes/manifests# 

kubectl apply -f .\manual-schedule-pod1.yaml
pod/my-nodename1 created

# Now the node handled scheduling the pod, and created in give NodeName.

Name:             my-nodename1
Namespace:        default
Priority:         0
Service Account:  default
Node:             surya-worker/172.18.0.4

kubectl get pods -o wide
NAME           READY   STATUS    RESTARTS   AGE   IP           NODE            NOMINATED NODE   READINESS GATES
my-nodename    1/1     Running   0          18m   10.244.1.2   surya-worker2   <none>           <none>
my-nodename1   1/1     Running   0          11m   10.244.2.2   surya-worker    <none>           <none>

# kube-scheduler.yaml pasted in /etc/kubernetes/manifest/  Now deployment of pods are handled by schedular 
root@surya-control-plane:/etc/kubernetes/manifests# ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml
root@surya-control-plane:/etc/kubernetes/manifests# mv /etc/k
kernel/              keyutils/            kube-scheduler.yaml  kubelet              kubernetes/
root@surya-control-plane:/etc/kubernetes/manifests# mv /etc/kube-scheduler.yaml .
root@surya-control-plane:/etc/kubernetes/manifests# ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml

# Following commands are usefull -

     - kuberctl apply -f name.yaml
     - docker ps -a
     - docker exec -it container_ID bash : login in control plane
     - pwd /etc/kubernetes/manifest/kube-scheduler.yaml  
     - kubectl get pods -o wide
     - kubectl desrcibe pod/pod-name { get details of pod}
     - Deploy lables and selectors according to label and selector: matchLabel :