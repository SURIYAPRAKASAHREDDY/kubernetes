
Health probes in Kubernetes are mechanisms to check the health of your applications running in containers. Kubernetes provides two types of health probes to ensure that the application is functioning properly and is accessible:

---

### 1. **Liveness Probe**
   - **Purpose:** Checks if the application is running. If the liveness probe fails, Kubernetes assumes the application is stuck or in an unrecoverable state, and it restarts the container.
   - **Example Use Case:** An application stuck in a deadlock that cannot process further requests.

---

### 2. **Readiness Probe**
   - **Purpose:** Checks if the application is ready to serve requests. If the readiness probe fails, the pod is removed from the service's endpoints, meaning it won't receive traffic until it's ready again.
   - **Example Use Case:** Waiting for an application to load necessary configurations or dependencies during startup.

---

### 3. **Startup Probe** (Introduced in Kubernetes 1.16)
   - **Purpose:** Designed specifically for applications with a long initialization phase. It checks whether the application has started successfully. If the probe fails, Kubernetes kills the container and restarts it.
   - **Example Use Case:** Applications that require a long warm-up time before becoming healthy.

---

### Types of Probes
Kubernetes supports three types of probes that can be used for liveness, readiness, and startup checks:

#### 1. **HTTP GET Probe**
   - Makes an HTTP GET request to the application and expects a 2xx or 3xx response.
   - **Example:**
     ```yaml
     livenessProbe:
       httpGet:
         path: /healthz
         port: 8080
       initialDelaySeconds: 3
       periodSeconds: 5
     ```

#### 2. **TCP Socket Probe**
   - Checks if a TCP connection can be established on a specified port.
   - **Example:**
     ```yaml
     readinessProbe:
       tcpSocket:
         port: 3306
       initialDelaySeconds: 3
       periodSeconds: 10
     ```

#### 3. **Command Probe (Exec Probe)**
   - Executes a command inside the container and checks the exit status (0 = success, non-zero = failure).
   - **Example:**
     ```yaml
     livenessProbe:
       exec:
         command:
           - cat
           - /tmp/healthy
       initialDelaySeconds: 5
       periodSeconds: 10
     ```

---

### Key Parameters
| Parameter               | Description                                                             |
|-------------------------|-------------------------------------------------------------------------|
| **`initialDelaySeconds`** | Time to wait after the container starts before performing the probe.   |
| **`periodSeconds`**      | Frequency of the probe (time interval between each probe).             |
| **`timeoutSeconds`**     | Timeout for each probe to complete.                                    |
| **`successThreshold`**   | Minimum consecutive successes required to consider the probe healthy.  |
| **`failureThreshold`**   | Number of consecutive failures before marking the probe as failed.     |

---

### Common Scenarios
1. **Web Server Application:**
   - Liveness Probe: Checks if the server responds on `/healthz`.
   - Readiness Probe: Verifies the server is ready to serve traffic.
   - Startup Probe: Ensures the server initializes properly.

   ```yaml
   livenessProbe:
     httpGet:
       path: /healthz
       port: 80
     initialDelaySeconds: 3
     periodSeconds: 5
   readinessProbe:
     httpGet:
       path: /readiness
       port: 80
     initialDelaySeconds: 5
     periodSeconds: 10
   ```

2. **Database Pod:**
   - Readiness Probe: Checks TCP socket availability on the database port.
   - Liveness Probe: Verifies the database process is alive using a command.

---

### Best Practices
1. **Separate Liveness and Readiness Probes:**
   - Avoid using the same endpoint for both; readiness checks should focus on availability, while liveness ensures the container is not stuck.
2. **Start with Conservative Values:**
   - Use generous `initialDelaySeconds` and `timeoutSeconds` to avoid false negatives, especially for applications with a slow startup.
3. **Keep Probes Lightweight:**
   - Avoid complex logic or heavy commands in probes to minimize resource consumption.


PS C:\Users\karri\OneDrive\Desktop\kind\kubernetes\probes> kubectl get pods
NAME            READY   STATUS    RESTARTS        AGE
liveness-exec   1/1     Running   6 (2m34s ago)   10m
PS C:\Users\karri\OneDrive\Desktop\kind\kubernetes\probes> kubectl get pods
NAME            READY   STATUS             RESTARTS      AGE
liveness-exec   0/1     CrashLoopBackOff   7 (38s ago)   12m

 kubectl get pods
NAME            READY   STATUS    RESTARTS   AGE
liveness-exec   1/1     Running   0          45s

kubectl exec -it liveness-exec -- sh
/ # command terminated with exit code 137

kubectl exec -it liveness-exec -- sh
/ # cd /healthz
sh: cd: can't cd to /healthz
/ # ls
bin           etc           lib           linuxrc       mnt           proc          root          sbin          tmp           var
dev           home          lib64         media         opt           product_uuid  run           sys           usr
/ # cd tmp
/tmp # ls
ldconfig     resolv.conf  secrets
/tmp # cat resolv.conf
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
/tmp # cd secrets/
/tmp/secrets # ls
kubernetes.io
/tmp/secrets # cat kubernetes.io/
cat: read error: Is a directory
/tmp/secrets # cd kubernetes.io/
/tmp/secrets/kubernetes.io # ls
serviceaccount
/tmp/secrets/kubernetes.io # cd serviceaccount/
/tmp/secrets/kubernetes.io/serviceaccount # ls
ca.crt     namespace  token
/tmp/secrets/kubernetes.io/serviceaccount # cat namespace command terminated with exit code 137
PS C:\Users\karri\OneDrive\Desktop\kind\kubernetes\probes> 


Here is livenessProbe is working and restarting on server failures.

we have passed a command that the /healthz remove. once it is removed application going to down, then livenessProbe is restarting the pod.

LivenessProbe -----> Running application --> is there any issue in application --> Liveness probe identify by httpGet and Restart ---> Live Application -----> if the probelm repeat again it will restart server up to restart limit. once restart limit reached it will go to unreached and get error like CrashLoopBackOff  error etc...

# ReadinessProbe

refer livenessandreadiness-http.yaml


kubectl get pods -o wide --watch
NAME    READY   STATUS    RESTARTS      AGE     IP            NODE            NOMINATED NODE   READINESS GATES
hello   0/1     Running   1 (69s ago)   2m24s   10.244.2.14   surya-worker2   <none>           <none>
hello   0/1     Running   2 (5s ago)    2m35s   10.244.2.14   surya-worker2   <none>           <none>
hello   1/1     Running   2 (20s ago)   2m50s   10.244.2.14   surya-worker2   <none>           <none>
hello   0/1     Running   2 (50s ago)   3m20s   10.244.2.14   surya-worker2   <none>           <none>


-  get pods -o wide --watch
- kubectl describe po/hello
- kubectl get pods

refer - probe-explain.yaml


# flow

**Refined Statement:**

Startup Probe → Liveness Probe → Readiness Probe

The Startup Probe checks if the application within the pod has successfully started. This is particularly useful for applications with slow or legacy initialization. During this phase, Kubernetes delays running the Liveness Probe and Readiness Probe until the Startup Probe succeeds.
Once the Startup Probe confirms the application is up and running:

The Liveness Probe ensures that the application remains healthy and restarts the pod if it becomes unresponsive or enters an unhealthy state.
The Readiness Probe verifies whether the application is ready to serve traffic and updates the pod's readiness status accordingly.
If the Startup Probe fails with an error (e.g., the application fails to initialize), Kubernetes considers the pod unhealthy and restarts it. Once restarted, the Readiness Probe will work with the new pod to ensure it's ready to route traffic.