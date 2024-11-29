## Create a multi-container pod.

# Init Containers in Kubernetes
An init container is a special type of container in Kubernetes that runs before the main application containers start. It is specifically designed to handle tasks that must complete before the main containers begin.

Key Features of Init Containers:
Purpose: They perform initialization tasks, such as:

Setting up configurations.
Pulling necessary data or secrets.
Waiting for a service dependency to become available.
Dependency Handling:

Init containers ensure that the environment is properly configured or the required resources are ready before the main containers start running.
They run sequentially, one after the other, and all must finish successfully before the main container starts.
# Creating cluster
kind create cluster --config config.yaml --name surya
Creating cluster "surya" ...
 • Ensuring node image (kindest/node:v1.31.2) 🖼  ...
 ✓ Ensuring node image (kindest/node:v1.31.2) 🖼
 • Preparing nodes 📦 📦   ...
 ✓ Preparing nodes 📦 📦 
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


kubectl logs my-app -n surya
Defaulted container "my-app" out of: my-app, init-myservice (init)
Error from server (BadRequest): container "my-app" in pod "my-app" is waiting to start: PodInitializing

Init container need to run 1st, then app pod will up and run. 

BusyBox is a single binary that provides a collection of Unix utilities. It's often referred to as the "Swiss Army Knife" of embedded Linux.


kubectl delete po/my-app -n surya
pod "my-app" deleted