# kubernetes
Kubernetes - concepts


In Kubernetes, RC, RS, NodePort, ClusterIP, LoadBalencing and Service (SVC) are components used to manage pods, ensure scalability, and expose applications running in your cluster to external users. Here's an overview of each:

### 1. RC (ReplicationController)
The ReplicationController (RC) is an older Kubernetes resource that ensures a specified number of pod replicas are running at any given time. If a pod fails or is deleted, the RC automatically creates a new one to maintain the desired number of replicas. However, Deployment has now largely replaced ReplicationController for managing applications.

# Key Points:
- Goal: Maintain a stable set of replica pods running at any given time.
- Use Case: Ensuring high availability for applications.
- Deprecation: ReplicationControllers are deprecated in favor of Deployments (introduced in Kubernetes 1.2).
 refer - rc.yaml 

### 2. RS (ReplicaSet)
A ReplicaSet is a newer, more advanced version of ReplicationController. It ensures that a specified number of replicas of a pod are running at all times. ReplicaSets are typically managed by a Deployment, which provides declarative updates to Pods and ReplicaSets.

# Key Points:
- Goal: Ensures a specified number of identical pods are running.
- Use Case: Mostly used indirectly through Deployments for scaling applications.
- Difference from RC: ReplicaSets support label selectors more flexibly and allow more complex use cases.
refer - rs.yaml

### 3. NodePort
A NodePort is a type of Kubernetes Service that exposes an application running inside the Kubernetes cluster to the external network. It opens a specific port on every node in the cluster and forwards the traffic to the appropriate pod. This is useful for exposing services outside of the Kubernetes cluster.

# Key Points:
- Goal: Expose a service running inside the cluster to the outside world by opening a static port on all nodes.
- Use Case: Allow external access to services without needing a load balancer.
- Ports: You specify a port in the `nodePort` field (usually between 30000-32767).
refer - nodeport.yaml

 **NOTE : node ports range 30000-32767**

In this example:
- The service `my-service` will be exposed on port `30001` on all nodes in the cluster.
- The service will forward traffic on port `80` to the pods on port `8080`.

# 4. SVC (Service)
A Service (SVC) is an abstraction that defines a logical set of pods and enables access to them. Services allow you to expose your pods, providing a stable endpoint (DNS name or IP) for accessing your application, even as the pods themselves may change or be scaled.

## Types of Services:
- ClusterIP (default): Exposes the service on an internal IP in the cluster (can only be accessed from within the cluster).
- NodePort: Exposes the service on a specific port on each node, which can be accessed externally.
- LoadBalancer: Exposes the service through a cloud provider’s load balancer (automatically provisions an external load balancer if the cluster is on a supported cloud provider).
- ExternalName: Maps a service to an external DNS name.

# Key Points:
- Goal: Provide stable, reliable networking to a set of pods, abstracting away the complexity of pod IP addresses and ensuring that service consumers can connect to pods regardless of changes.
- Use Case: Exposing your app internally or externally via DNS.


# Relationships Between RC, RS, NodePort, and SVC:
- ReplicationController (RC) and ReplicaSet (RS) manage the pods by ensuring the desired number of replicas are running.
- Service (SVC) is used to expose applications running inside the cluster to the outside world or other services within the cluster. A NodePort type service is one way to expose the service to the external network.

# Summary:
- RC (ReplicationController): Ensures the desired number of replicas of pods.
- RS (ReplicaSet): A more advanced version of RC, typically managed by Deployments.
- NodePort: Exposes a service on a static port on each node of the cluster, allowing external access.
- SVC (Service): Exposes a set of pods internally or externally through various types like ClusterIP, NodePort, and LoadBalancer.