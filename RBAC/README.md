# RBAC :

# AUTHENTICATION                                                   AUTHORIZATION
  Who are you ?                                                     what you can do ?
-> symmetric key encryption                                     - RBAC
-> Asymmetric key encryption                                    - ABAC
-> Certificate                                                  - NODE
                                                                - WEBHOOKS 

- RBAC :  In RBAC, we are creating Roles, this roles we are assigning to users/groups, By using roleBinding.
- NODE :  This Node is work in nodes level, example API server Intarct with KUBELET.
- Webhook : webhooks are third party systems, and Open policy agents, access managements <OPA>
- ABAC : Attribute based Authentication Control, here we are Setting permission and policies in API Server. Once we are added new users the API server need to restart to apply. We need to add new users on require basis, So every time is not possible to restart API server.

**Role Based Access Control** 

Kubernetes **Authentication** and **Authorization** are key components that ensure secure access to the cluster resources. Here's a simplified breakdown of how they work:

---

## 1. **Authentication**: *Who are you?*  
Authentication in Kubernetes identifies **who** is making the request (users, service accounts, kubelets, or API clients).

### **Authentication Methods**:
Kubernetes does not manage users directly (like usernames/passwords). Instead, it supports the following authentication mechanisms:

| Method                  | Description                                                             |
|-------------------------|-------------------------------------------------------------------------|
| **Client Certificates** | TLS certificates (public/private key pairs) for client authentication.  |
| **Bearer Tokens**       | Tokens (e.g., JWT) passed with API requests for user verification.      |
| **Service Accounts**    | Tokens automatically created for Pods to authenticate within the cluster. |
| **OIDC (OpenID Connect)** | Integrates with identity providers like Google, Azure AD, Keycloak, etc. |
| **Webhook Authentication** | Custom external authentication using webhook API calls.              |

---

### **How Authentication Works**:
1. A request (e.g., from `kubectl`, a client, or a Pod) is sent to the **API server**.
2. The API server checks the request headers for authentication credentials.
3. Based on the credentials, the API server identifies:
   - The **user** (e.g., `john`, `admin`)  
   - The **group** (e.g., `system:masters`, `dev-team`)  
4. If authentication fails, the request is **rejected**.

---

## 2. **Authorization**: *What are you allowed to do?*  
After authenticating the user, Kubernetes determines **what actions** they can perform using **Authorization**.

### **Authorization Modes**:
Kubernetes supports several authorization mechanisms:

| Mode                                       |         Description                                                    |
|--------------------------------------------|------------------------------------------------------------------------|
| **RBAC (Role-Based Access Control)**       | Assigns permissions to users based on **roles** and **bindings**.      |
| **ABAC (Attribute-Based Access Control)**  | Uses policies defined in a file to authorize requests.                 |
| **Node Authorization**                     | Grants kubelets access to resources on nodes.                          |
| **Webhook Authorization**                  | For custom external authorization logic via a webhook API.             |

---

### **Role-Based Access Control (RBAC)**  
RBAC is the most widely used authorization method in Kubernetes.

1. **Roles and ClusterRoles**:
   - **Role**: Grants permissions **within a namespace**.
   - **ClusterRole**: Grants permissions cluster-wide.

   Example Role (read-only access to pods in a namespace):
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     name: pod-reader
     namespace: surya
   rules:
   - apiGroups: [""]
     resources: ["pods"]
     verbs: ["get", "list", "watch"]
   ```

2. **RoleBinding and ClusterRoleBinding**:
   - **RoleBinding**: Associates a Role with users or service accounts **in a namespace**.
   - **ClusterRoleBinding**: Associates a ClusterRole with users or service accounts **across the cluster**.

   Example RoleBinding:
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: read-pods-binding
     namespace: surya
   subjects:
   - kind: User
     name: john
   roleRef:
     kind: Role
     name: pod-reader
     apiGroup: rbac.authorization.k8s.io
   ```

### **How Authorization Works**:
1. After authenticating the user, the API server determines if the user is **authorized** to perform the requested action.
2. For RBAC:
   - The API server checks if there’s a **Role** or **ClusterRole** granting the necessary permission.
   - If a RoleBinding/ClusterRoleBinding matches, the request is allowed.

---

## **Simplified Flow of a Request**:

1. **Authentication**:  
   "Who are you?"  
   - The API server verifies user credentials (e.g., tokens, certificates).  

2. **Authorization**:  
   "What are you allowed to do?"  
   - The API server checks RBAC/other rules to determine if the user has the required permissions.

3. **Admission Control** (Optional):  
   - Final checks are done (e.g., validating policies, resource quotas).

If all checks pass, the request is allowed; otherwise, it’s rejected.

---

## **Example Workflow**
- User `john` runs:
  ```bash
  kubectl get pods --namespace=surya
  ```

- **Authentication**:  
   Kubernetes API server checks the client’s **certificate** or **token** to authenticate `john`.

- **Authorization**:  
   The API server checks if `john` has the **RoleBinding** or **ClusterRoleBinding** granting access to:
   - `resource: pods`
   - `verb: get`
   - `namespace: default`

- **Result**:  
   If the conditions match, `john` is allowed to list pods; otherwise, access is denied.

---

### **Key Takeaways**
1. **Authentication**: Identifies "who you are" using certificates, tokens, or external providers (OIDC, Webhook).
2. **Authorization**: Determines "what you can do" using RBAC, ABAC, or other methods.
3. RBAC is the most common authorization method in Kubernetes due to its flexibility and simplicity.

------------------------------------------------------------------------------------------
# API SERVER YAML 

```
# cat kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 172.18.0.2:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=172.18.0.2
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --runtime-config=
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/16
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: registry.k8s.io/kube-apiserver:v1.31.2
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 172.18.0.2
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-apiserver
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: 172.18.0.2
        path: /readyz
        port: 6443
        scheme: HTTPS
      periodSeconds: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 250m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 172.18.0.2
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
  hostNetwork: true
  priority: 2000001000
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
status: {}

```

Please refere rabc-manifest.yaml for few examples.

# below commands are help here :

 - kubectl get nodes
 - docker ps -a
 - docker exec -it 8c4f3df0627c -- sh   # find mistake please 
 - kubectl get pod -n surya
 - kubectl auth  whoami
 - kubectl get roles
 - kubectl get role -n kube-system
 - kubectl apply -f .\role.yaml
 - kubectl get role -n surya
 - kubectl describe role pod-reader -n surya
 - kubectl apply -f .\roleBinding.yaml
 - kubectl get rolebinding -n surya
 - k auth whoami
 - kubectl auth can-i get pod --as surya