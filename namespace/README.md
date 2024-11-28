# kubernetes
Kubernetes - concepts


Communicating 2 pods and 2 services by using fqdn

Create two namespaces and name them ns1 and ns2
# ns1.yaml and ns2.yaml
# kubectl get namespace  -- create new namespace
# kuberctl apply -f ns1.yaml - to create pod ns1
# kuberctl apply -f ns2.yaml - to create pod ns2
# kubectl get po -n surya   { -n namespace: it will display under same namespace}


Create a deployment with a single replica in each of these namespaces with the image as nginx and name as deploy-ns1 and deploy-ns2, respectively

# kubectl explain deployments


### kubectl get all -n surya
NAME                                 READY   STATUS    RESTARTS   AGE
pod/deploymentns1-cc44fbf6c-hcszc    1/1     Running   0          8m9s
pod/deploymentns2-5c6dd65955-r5x7f   1/1     Running   0          4s
pod/ns1                              1/1     Running   0          25m
pod/ns2                              1/1     Running   0          21m

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/deploymentns1   1/1     1            1           8m9s
deployment.apps/deploymentns2   1/1     1            1           4s

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/deploymentns1-cc44fbf6c    1         1         1       8m9s
replicaset.apps/deploymentns2-5c6dd65955   1         1         1       4s



  46 kubectl get pods   : To view the pods
  47 kubectl get pods -n surya  : To view the pods under the ns surya
  48 kubectl get deploy -n surya  :  To view the deployments under the ns surya 
  53 kubectl get all  : To See All Services , Pods, ReplicaSet, rc And  Deployments
  55 kubectl get rs  : To See ReplicaSet 

   # NOTE : Here we are not created any RS but if your created Deployments the rs will create. follow the architeture.

  <!-- --------------------------------------
   Client 
     |
   Deployments
     |
   ReplicaSet = 1   //      ReplicaSet 3
      |                        |
   Pod =1          //        Pod = 3

   ------------------------------------ -->
  
  ## 56 kubectl get rs -n surya
  58 kubectl describe depoly/deploymentns1   : To Describe about Deployments
  59 kubectl get all
  60 kubectl get deploy -n surya
  61 kubectl describe deploy/deploymentns1
  62 kubectl describe deploy/deploymentns1 -n surya
  63 kubectl get all -n surya
  64 kubectl apply -f .\deployment2.yaml
  65 kubectl get all -n surya

Get the IP address of each of the pods 

# kubectl get pods -o wide -n surya
NAME                             READY   STATUS    RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
deploymentns1-cc44fbf6c-hcszc    1/1     Running   0          20m   10.244.1.*   kind-worker   <none>           <none>
deploymentns2-5c6dd65955-r5x7f   1/1     Running   0          12m   10.244.1.*   kind-worker   <none>           <none>
ns1                              1/1     Running   0          38m   10.244.1.*   kind-worker   <none>           <none>
ns2                              1/1     Running   0          34m   10.244.1.*   kind-worker   <none>           <none>

Exec into the pod of deploy-ns1 and try to curl the IP address of the pod running on deploy-ns2

# kubectl exec -it deploymentns1-cc44fbf6c-hcszc -n surya -- bash

root@deploymentns1-cc44fbf6c-hcszc:/etc# cat resolv.conf 
search surya.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
root@deploymentns1-cc44fbf6c-hcszc:/etc# 



Your pod-to-pod connection should work, and you should be able to get a successful response back.
Now scale both of your deployments from 1 to 3 replicas.
Create two services to expose both of your deployments and name them svc-ns1 and svc-ns2
exec into each pod and try to curl the IP address of the service running on the other namespace.
This curl should work.
Now try curling the service name instead of IP. You will notice that you are getting an error and cannot resolve the host.
Now use the FQDN of the service and try to curl again, this should work.
In the end, delete both the namespaces, which should delete the services and deployments underneath them.