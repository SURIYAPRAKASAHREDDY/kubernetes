
In Kubernetes, **ConfigMaps** are used to store configuration data in key-value pairs. They can be categorized based on how the data is created and used. Here’s an overview of the **types of ConfigMaps**:

---

### **1. Based on Creation Method**

#### **a. ConfigMap from Literal Values**
- Created directly by specifying key-value pairs in the command.

- Example:
  ```bash
  kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2
  ```
- Use Case: Simple configurations with a few key-value pairs.

---

#### **b. ConfigMap from a File**
- Created by reading configuration data from a file.
- Example:
  ```bash
  kubectl create configmap my-config --from-file=config.txt
  ```
  If `config.txt` contains:
  ```
  key1=value1
  key2=value2
  ```
- Use Case: When configuration data is already stored in files.

---

#### **c. ConfigMap from a Directory**
- Created by reading all files in a directory as individual keys (file names become keys, and file contents become values).
- Example:
  ```bash
  kubectl create configmap my-config --from-file=/path/to/directory/
  ```
- Use Case: Managing configurations for multiple components stored in different files.

---

#### **d. ConfigMap from YAML/JSON Manifest**
- Created by defining the ConfigMap in a YAML or JSON file and applying it with `kubectl`.
- Example `configmap.yaml`:
  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: my-config
  data:
    key1: value1
    key2: value2
  ```
  Command:
  ```bash
  kubectl apply -f configmap.yaml
  ```
- Use Case: Preferred for version-controlled configurations.

---

### **2. Based on Usage in Applications**

#### **a. Environment Variable ConfigMap**
- ConfigMap data is injected as environment variables into containers.
- Example in a Pod YAML:
  ```yaml
  env:
    - name: CONFIG_KEY
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: key1
  ```
- Use Case: Dynamically configuring applications without modifying container images.

---

#### **b. Mounted Volume ConfigMap**
- ConfigMap is mounted as a volume in the container filesystem.
- Example:
  ```yaml
  volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: my-config
  ```
- Use Case: Providing configuration files directly to applications.

---

#### **c. Command-Line Arguments**
- ConfigMap values can be passed as command-line arguments to the container's entrypoint.
- Example:
  ```yaml
  args:
    - --config=$(CONFIG_KEY)
  env:
    - name: CONFIG_KEY
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: key1
  ```
- Use Case: Applications that take configuration via CLI flags.

---

#### **d. Immutable ConfigMap**
- Created as an immutable resource to prevent accidental updates after creation.
- Example:
  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: immutable-config
  immutable: true
  data:
    key1: value1
  ```
- Use Case: Ensuring stability of configurations in production environments.

---

### Summary Table:

| **Type**                      | **Creation Method/Usage**                                     | **Use Case**                                      |
|-------------------------------|-------------------------------------------------------------|--------------------------------------------------|
| Literal                       | `--from-literal`                                             | Quick setup for simple key-value pairs.          |
| File-Based                    | `--from-file=config.txt`                                     | When config is stored in files.                  |
| Directory-Based               | `--from-file=/path/to/dir`                                   | Managing multiple configurations.                |
| YAML/JSON Manifest            | `kubectl apply -f`                                           | Version-controlled, complex setups.             |
| Environment Variable          | Inject data as `env` variables                              | Applications requiring dynamic environment vars. |
| Mounted Volume                | Mount ConfigMap as a volume                                 | Apps needing file-based configurations.          |
| Command-Line Arguments        | Pass ConfigMap as CLI flags                                 | Apps configurable via CLI.                       |
| Immutable ConfigMap           | `immutable: true`                                           | Prevent accidental changes.                      |

---

## Method	Use Case

valueFrom.configMapKeyRef	When you need specific keys from the ConfigMap.
envFrom.configMapRef	When you need to inject all keys/values from the ConfigMap as environment variables.



# Creating cluster 


kubernetes> kind create cluster --name surya --config .\config.yaml

Creating cluster "surya" ...
 • Ensuring node image (kindest/node:v1.31.2) 🖼  ...
 ✓ Ensuring node image (kindest/node:v1.31.2) 🖼
 • Preparing nodes 📦 📦 📦   ...
 ✓ Preparing nodes 📦 📦 📦 
 • Writing configuration 📜  ...
 ✓ Writing configuration 📜
 • Starting control-plane 🕹️  ...
 ✓ Starting control-plane 🕹️
 • Installing CNI 🔌  ...
 ✓ Installing CNI 🔌
 • Installing StorageClass 💾  ...
 ✓ Installing StorageClass 💾
 • Joining worker nodes 🚜  ...
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-surya"
You can now use your cluster with:

kubectl cluster-info --context kind-surya

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/

kubernetes> kubectl get nodes
NAME                  STATUS   ROLES           AGE     VERSION
surya-control-plane   Ready    control-plane   4m9s    v1.31.2
surya-worker          Ready    <none>          3m56s   v1.31.2
surya-worker2         Ready    <none>          3m56s   v1.31.2


Configmaps and Secrets> kubectl create ns surya 
namespace/surya created
Configmaps and Secrets> kubectl apply -f .\cm.yaml
pod/my-app created
Configmaps and Secrets> kubectl get pods
No resources found in default namespace.
Configmaps and Secrets> kubectl get pod -n surya
NAME     READY   STATUS    RESTARTS   AGE
my-app   1/1     Running   0          11s
Configmaps and Secrets> kubectl exec my-app -n surya
error: you must specify at least one command for the container
Configmaps and Secrets> kubectl exec -it my-app -n surya -- sh
/ # ls
bin           dev           etc           home          proc          product_uuid  root          sys           tmp           usr           var
/ # echo $FIRSTNAME
surya


# TEST EM-ENV.YAML  - passing multiple veriables 

Configmaps and Secrets> kubectl apply -f .\cm-env.yaml
pod/my-app-env created


Configmaps and Secrets> 
NAME         READY   STATUS    RESTARTS   AGE
my-app       1/1     Running   0          12m
my-app-env   1/1     Running   0          10s


Configmaps and Secrets> kubectl exec -it my-app-env -n surya -- sh
/ # echo $FIRSTNAME
surya
/ # echo $PASSWORD
surya
/ # echo $CM
ConfigMap


###################################################

Inject configMap to pod


kubernetes\Configmaps and Secrets> kubectl create cm test-cm --from-literal=firstname=Surya --from-literal=newpwd=surya@123
- configmap/test-cm created

kubernetes\Configmaps and Secrets> kubectl describe cm/test-cm

Name:         test-cm
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
firstname:
----
Surya
newpwd:
----
surya@123

BinaryData
====

Events:  <none>


kubernetes\Configmaps and Secrets>


Method	Use Case

valueFrom.configMapKeyRef	When you need specific keys from the ConfigMap.
envFrom.configMapRef	When you need to inject all keys/values from the ConfigMap as environment variables.

refere cm-config.yaml for multiple references. 

# Now we are looking at Volume mount config and secrets configaration.

 Kindly understand this yaml. i writen all in one place. i will explain below.


 kubernetes\Configmaps and Secrets> kubectl apply -f .\cm-volume.yaml
configmap/app-config created
pod/combined-volume-pod created

- Password should be base64

Error from server (BadRequest): error when creating ".\\cm-volume.yaml": Secret in version "v1" cannot be handled as a Secret: illegal base64 data at input byte 4


kubernetes\Configmaps and Secrets> kubectl apply -f .\cm-volume.yaml
configmap/app-config unchanged
secret/db-credentials created
pod/combined-volume-pod configured

- Output thru mount volume, here we are not passing any key values so, we need to add key reference.

kubernetes\Configmaps and Secrets> 


 APP_ENV=$(cat /etc/config/app.properties | grep APP_ENV | cut -d '=' -f2)
/etc/config # echo $APP_ENV
production
/etc/config #


- Key value reference both in same yaml.

---

apiVersion: v1
kind: Pod
metadata:
  name: combined-pod
spec:
  containers:
    - name: app-container
      image: busybox
      command: ["/bin/sh", "-c", "echo $APP_ENV && APP_DEBUG=$(cat /etc/config/app.properties | grep APP_DEBUG | cut -d '=' -f2) && echo $APP_DEBUG && sleep 3600"]
      env:
        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_ENV
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
          readOnly: true
  volumes:
    - name: config-volume
      configMap:
        name: app-config
---

in side :


kubectl exec -it combined-volume-pod -n surya -- sh
/ # cd /etc/
/etc # ls
config         hostname       localtime      nsswitch.conf  resolv.conf    shadow
group          hosts          network        passwd         secrets
/etc # cd config/
/etc/config # ls
app.properties
/etc/config # ls -
ls: -: No such file or directory
/etc/config # ls -a
.                                 ..2024_12_16_05_49_16.1039036185  app.properties
..                                ..data
/ # cd /etc/
/etc # ls
config         hostname       localtime      nsswitch.conf  resolv.conf    shadow
group          hosts          network        passwd         secrets
/etc # cd config/
/etc/config # ls
app.properties
/etc/config # cat app.properties
APP_ENV=production
APP_DEBUG=false
/etc/config # cat /etc/secrets/username
my_username/etc/config # ^C

/etc/config # cat /etc/config/app.properties | grep APP_ENV | cut -d '=' -f2
production
/etc/config # cat /etc/config/app.properties | grep APP_ENV 
APP_ENV=production
/etc/config # echo $APP_ENV

/etc/config # echo $ APP_ENV
$ APP_ENV
/etc/config # echo "The current environment is: $APP_ENV"

The current environment is: 
/etc/config # APP_ENV=$(cat /etc/config/app.properties | grep APP_ENV | cut -d '=' -f2)
/etc/config # echo $APP_ENV
production

# manifest.yaml for both Secrets, configMap 

Below is the commands are used here:

   -  kubectl create cm app-cm  --from-literal=firstname=surya
   - kubectl get cm
   - kubectl get cm app-cm
   - kubectl describe cm/app-cm
   - kubectl delete cm/app-cm
   - kubectl create cm test-cm --from-literal=firstname=Surya
   - kubectl get pods
   - kubectl create cm test-cm --from-literal=firstname=Surya --from-literal=newpwd=surya¾123
   - kubectl exec -it test -n surya -- sh
   - kubectl exec -it combined-volume-pod -n surya -- sh
   - /etc/config/app.properties | grep APP_ENV | cut -d '=' -f2
