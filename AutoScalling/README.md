**Autoscaling in Kubernetes** is a feature that automatically adjusts the number of pods, replicas, or nodes in a cluster based on resource utilization or custom metrics. 
It ensures applications can handle varying loads effectively while optimizing resource usage. Kubernetes offers three types of autoscaling:

---

### **1. Horizontal Pod Autoscaler (HPA)**
**HPA** scales the number of pods in a deployment, replica set, or stateful set based on observed metrics like CPU or memory utilization, or custom metrics.

#### How It Works:
- HPA continuously monitors resource usage (e.g., CPU, memory) and compares it against predefined thresholds.
- If resource usage exceeds or drops below the threshold, HPA increases or decreases the number of pods.

#### Configuration Example:
- Define an HPA object:

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-example
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```
- **Explanation**:
  - Minimum pods: 2.
  - Maximum pods: 10.
  - If average CPU utilization across pods exceeds 50%, scale up.

#### Key Points:
- Requires **Metrics Server** to be installed for resource metrics (CPU/Memory).
- Supports custom metrics using the **Custom Metrics API** or **External Metrics**.

---

### **2. Vertical Pod Autoscaler (VPA)**
**VPA** adjusts the resource requests and limits (CPU/memory) of a pod based on actual usage patterns. Unlike HPA, it doesn't change the number of pods.

#### How It Works:
- VPA analyzes pod resource usage over time.
- It recommends or directly adjusts the pod's resource requests/limits to ensure optimal performance.

#### Configuration Example:
- Define a VPA object:
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: vpa-example
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"
```
- **Modes**:
  - `Off`: Only provides recommendations.
  - `Auto`: Automatically applies changes (may cause pod restarts).
  - `Initial`: Applies recommendations only during pod creation.

#### Key Points:
- Useful for optimizing resource allocation and minimizing over-provisioning.
- May restart pods if resource requests/limits are updated.

---

### **3. Cluster Autoscaler**
**Cluster Autoscaler** adjusts the number of nodes in a cluster based on pod scheduling needs. It's commonly used with cloud providers like AWS, GCP, or Azure.

#### How It Works:
- If pods cannot be scheduled due to insufficient node resources, Cluster Autoscaler adds nodes.
- If nodes become underutilized (e.g., running only system pods), Cluster Autoscaler removes nodes.

#### Configuration:
- Enable Cluster Autoscaler for your cloud-managed Kubernetes cluster.
- Example command (GCP):
  ```bash
  gcloud container clusters update my-cluster \
    --enable-autoscaling --min-nodes=1 --max-nodes=10 --zone=us-central1-a
  ```

#### Key Points:
- Works with node pools or instance groups.
- Requires proper labels and taints for nodes to align with pod requirements.

---

### **Choosing Between HPA, VPA, and Cluster Autoscaler**
| Feature                | HPA                           | VPA                       | Cluster Autoscaler            |
|------------------------|-------------------------------|---------------------------|-------------------------------|
| **Scale What**        | Number of pods               | Pod resource requests     | Number of nodes              |
| **When to Use**       | High/low traffic workloads   | Optimize resource usage   | Insufficient node resources  |
| **Impact**             | Fast, dynamic scaling        | May restart pods          | Adds/removes nodes           |

---

### **Combined Usage**
You can use HPA, VPA, and Cluster Autoscaler together for optimal scaling:
- Use **HPA** to adjust pod counts dynamically.
- Use **VPA** to tune pod resource requests/limits.
- Use **Cluster Autoscaler** to ensure sufficient nodes are available for the scaled pods.

---

Step 1 : Created a deployment and service pod : php-apache. and add HPA autoscale on pod, and put some load on it. 
Now your cpu getting high and HPA involue and create a new pods according to load, upto max replica.

once maximum it got failed. once load stopped/interputed then the HPA autoscale kill reduce the pod count.

Horizental Pod Autoscaling  --- >  1 pod running + **load** ---> HPA --> 2 pod ---> 3 pod -----> 4 pods || no Load 4 pods----> 3 pods ---> 2 --> 1 pod

Please refer hpa.yaml for hpa applying on deployment. And Metric-Server make sure up and ruinning.

 kubectl get pods -n kube-system
 kubectl apply -f .\metrics-server.yaml   : metrics-server-699f9ddd54-jbb68   

# kubectl get pods

NAME                         READY   STATUS              RESTARTS   AGE
php-apache-d87b7ff46-lbdrj   0/1     ContainerCreating   0          9s

AutoScalling> kubectl get service
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5m18s
php-apache   ClusterIP   10.96.56.4   <none>        80/TCP    18s

AutoScalling> kubectl get service
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   43m
php-apache   ClusterIP   10.96.56.4   <none>        80/TCP    38m



AutoScalling> kubectl autoscale deploy php-apache --cpu-percent=50 --min=1 --max=4
horizontalpodautoscaler.autoscaling/php-apache autoscaled

AutoScalling> kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
php-apache-d87b7ff46-lbdrj   1/1     Running   0          40m


AutoScalling> kubectl get hpa    

AutoScalling> kubectl get hpa
NAME         REFERENCE               TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   cpu: 0%/50%   1         4         1          7m12s

AutoScalling> kubectl get hpa
NAME         REFERENCE               TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   cpu: 109%/50%   1         4             2      
7m38s
AutoScalling> kubectl get hpa

NAME         REFERENCE               TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   cpu: 250%/50%   1         4         3          
7m51s

AutoScalling> kubectl get hpa
NAME         REFERENCE               TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   cpu: 181%/50%   1         4         4          
8m4s


AutoScalling> kubectl  get hpa --watch
NAME         REFERENCE               TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   cpu: 143%/50%   1         4         4          
8m30s
php-apache   Deployment/php-apache   cpu: 139%/50%   1         4         4          
8m31s
php-apache   Deployment/php-apache   cpu: 121%/50%   1         4         4          
8m46s
AutoScalling> kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
load-generator               1/1     Running   0          2m13s
php-apache-d87b7ff46-hmfdl   1/1     Running   0          89s
php-apache-d87b7ff46-lbdrj   1/1     Running   0          49m
php-apache-d87b7ff46-nd2qn   1/1     Running   0          74s
php-apache-d87b7ff46-wk6k9   1/1     Running   0          89s


AutoScalling> kubectl get hpa    
NAME         REFERENCE               TARGETS        MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   cpu: 89%/50%   1         4         4          9m9s

AutoScalling> kubectl  get hpa --watch
NAME         REFERENCE               TARGETS        MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   cpu: 85%/50%   1         4         4          9m20s
php-apache   Deployment/php-apache   cpu: 90%/50%   1         4         4          9m31s
php-apache   Deployment/php-apache   cpu: 80%/50%   1         4         4          9m46s


AutoScalling> kubectl  get hpa --watch
NAME         REFERENCE               TARGETS        MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   cpu: 80%/50%   1         4         4          9m48s


NAME         REFERENCE               TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   cpu: 0%/50%   1         4         4          11m


AutoScalling> kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
php-apache-d87b7ff46-hmfdl   1/1     Running   0          3m50s
php-apache-d87b7ff46-lbdrj   1/1     Running   0          51m
php-apache-d87b7ff46-nd2qn   1/1     Running   0          3m35s
php-apache-d87b7ff46-wk6k9   1/1     Running   0          3m50s



AutoScalling> kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
php-apache-d87b7ff46-lbdrj   1/1     Running   0          56m

AutoScalling> kubectl get hpa 
NAME         REFERENCE               TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   cpu: 0%/50%   1         4         1          15m


AutoScalling> kubectl get hpa php-apache -o yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  creationTimestamp: "2024-12-12T06:24:51Z"
  name: php-apache
  namespace: default
  resourceVersion: "3117"
  uid: abcce324-0f22-4d97-9f82-8f91c4594be8
spec:
  maxReplicas: 4
  metrics:
  - resource:
      name: cpu
      target:
        averageUtilization: 50
        type: Utilization
    type: Resource
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
status:
  conditions:
  - lastTransitionTime: "2024-12-12T06:25:06Z"
    message: recommended size matches current size
    reason: ReadyForNewScale
    status: "True"
    type: AbleToScale
  - lastTransitionTime: "2024-12-12T06:25:06Z"
    message: the HPA was able to successfully calculate a replica count from cpu resource
      utilization (percentage of request)
    reason: ValidMetricFound
    status: "True"
    type: ScalingActive
  - lastTransitionTime: "2024-12-12T06:39:52Z"
    message: the desired replica count is less than the minimum replica count
    reason: TooFewReplicas
    status: "True"
    type: ScalingLimited
  currentMetrics:
  - resource:
      current:
        averageUtilization: 0
        averageValue: 1m
      name: cpu
    type: Resource
  currentReplicas: 1
  desiredReplicas: 1
  lastScaleTime: "2024-12-12T06:39:52Z"
AutoScalling> history



# Please folow below commands 

 - kubectl get nodes
 - kind create cluster -n surya --config .\config.yaml
 - kubectl get nodes
- kubectl get pods -n kube-system
- kubectl apply -f .\deployment.yaml
- kubectl get service
**kubectl autoscale deploy php-apache --cpu-percent=50 --min=1 --max=4**
- kubectl get pods
- kubectl get hpa
- kubectl  get hpa --watch
- kubectl get hpa php=apache -o yaml

**kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"**  | O capital |