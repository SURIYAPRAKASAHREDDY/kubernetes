# CKA in 30 Posts

Thirty bite-sized posts for the **Certified Kubernetes Administrator (CKA)** exam — each works as a LinkedIn post for fellow candidates *and* as a spaced-revision card. One hook, one insight, one thing to remember.

> Part of *The CKA Deep Reference* study book. Accurate to Kubernetes 1.30+.

---

## Post 01 — The exam clock is your real opponent

The CKA isn't hard because of the concepts — it's hard because you have ~7 minutes per task. Hand-typing YAML from memory is how people run out of time. The move: generate a skeleton imperatively, then edit only what changes.

```bash
k run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
k create deploy web --image=nginx --replicas=3 \
  --dry-run=client -o yaml > deploy.yaml
```

**Remember:** `--dry-run=client -o yaml` turns kubectl into a YAML generator — never start from a blank file.

`#CKA`  `#Kubernetes`  `#kubectl`  `#DevOps`

---

## Post 02 — The 30-second setup that saves 30 minutes

Before you touch a single question, paste these into your shell. `k` saves thousands of keystrokes; `do` and `now` make dry-run and fast-delete one-word habits.

```bash
alias k=kubectl
export do="--dry-run=client -o yaml"   # k run x --image=nginx $do
export now="--force --grace-period=0"   # k delete pod x $now
```

**Remember:** `k run web --image=nginx $do > web.yaml` — muscle memory beats typing.

`#CKA`  `#Kubernetes`  `#Productivity`  `#SRE`

---

## Post 03 — CrashLoopBackOff is a symptom, not a diagnosis

Everyone panics at CrashLoopBackOff. But it just means "container keeps exiting, so I'm backing off before restarting." The *cause* is in the logs — a bad command, missing config, a failing app. The status only tells you the container won't stay up.

```bash
k logs pod --previous     # logs from the crashed instance
k describe pod pod        # Last State + Exit Code + Events
```

**Remember:** Don't restart the pod — read `--previous` logs and the Exit Code. Exit 1 = app error, 137 = OOMKilled.

`#CKA`  `#Kubernetes`  `#Troubleshooting`  `#SRE`

---

## Post 04 — Init containers vs sidecars

Both live in the Pod, but they run at different times. **Init containers** run to completion, one after another, *before* app containers start — perfect for setup, migrations, waiting on a dependency. A **sidecar** (a native init container with `restartPolicy: Always` since 1.29) runs *alongside* the app for its whole life — logging, proxies, config reload.

```bash
initContainers:
  - name: wait-db
    image: busybox
    command: ["sh","-c","until nc -z db 5432; do sleep 2; done"]
```

**Remember:** Init = runs then exits, blocks startup. Sidecar = runs forever, next to the app.

`#CKA`  `#Kubernetes`  `#Pods`  `#CloudNative`

---

## Post 05 — The three-layer chain you can undo

A Deployment doesn't manage Pods directly. **Deployment → ReplicaSet → Pods.** Each rollout creates a *new* ReplicaSet and scales the old one down — which is exactly why you can roll back in one command. The old ReplicaSet is still there at zero replicas, waiting.

```bash
k rollout status deploy/web
k rollout history deploy/web
k rollout undo deploy/web --to-revision=2
```

**Remember:** `rollout undo` works because Kubernetes keeps old ReplicaSets around — history is your safety net.

`#CKA`  `#Kubernetes`  `#Deployments`  `#DevOps`

---

## Post 06 — maxSurge and maxUnavailable control the blast radius

Rolling updates aren't magic — two numbers decide how aggressive they are. **maxSurge** = how many extra pods you can create above desired. **maxUnavailable** = how many you can drop below desired. Want zero downtime? maxUnavailable: 0. Want speed? Bump maxSurge.

```bash
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1          # can be a count or %
    maxUnavailable: 0    # never go below desired capacity
```

**Remember:** maxUnavailable: 0 + maxSurge: 1 = safest zero-downtime rollout.

`#CKA`  `#Kubernetes`  `#Deployments`  `#SRE`

---

## Post 07 — Requests schedule, limits throttle

The single most confused pair in Kubernetes. **Requests** are what the scheduler uses to find a node with enough room — they reserve capacity. **Limits** are the ceiling the runtime enforces at runtime. Exceed a CPU limit and you get throttled; exceed a memory limit and you get OOMKilled (Exit 137).

```bash
resources:
  requests: {cpu: "250m", memory: "128Mi"}   # used for scheduling
  limits:   {cpu: "500m", memory: "256Mi"}    # enforced at runtime
```

**Remember:** No node fits the *request* → Pod stays Pending. Exceed the memory *limit* → OOMKilled.

`#CKA`  `#Kubernetes`  `#Scheduling`  `#SRE`

---

## Post 08 — QoS decides who dies first

When a node runs out of memory, Kubernetes evicts pods in a strict order based on their QoS class — which you don't set directly, it's derived from your requests/limits. **Guaranteed** (requests == limits) is evicted last. **Burstable** (some set) is next. **BestEffort** (nothing set) is first out the door.

```bash
k get pod app -o jsonpath='{.status.qosClass}'   # Guaranteed | Burstable | BestEffort
```

**Remember:** Eviction order = BestEffort → Burstable → Guaranteed. Set requests == limits for your critical pods.

`#CKA`  `#Kubernetes`  `#QoS`  `#Reliability`

---

## Post 09 — Three probes, three jobs

They sound similar but answer different questions. **Liveness**: "is it stuck? restart it." **Readiness**: "can it take traffic? if not, pull it from the Service." **Startup**: "is it still booting? hold off the other two." Slow-starting apps need a startup probe or liveness will kill them mid-boot.

```bash
livenessProbe:  {httpGet: {path: /healthz, port: 8080}, periodSeconds: 10}
readinessProbe: {httpGet: {path: /ready,   port: 8080}, periodSeconds: 5}
startupProbe:   {httpGet: {path: /healthz, port: 8080}, failureThreshold: 30}
```

**Remember:** Liveness restarts, readiness gates traffic, startup buys boot time.

`#CKA`  `#Kubernetes`  `#Probes`  `#Reliability`

---

## Post 10 — base64 is not encryption

ConfigMaps and Secrets are almost identical objects — the difference is intent, not security. Secret values are base64-*encoded*, which anyone can decode in one command. Real protection comes from RBAC and enabling encryption-at-rest in etcd.

```bash
k create secret generic db --from-literal=pass=s3cret
k get secret db -o jsonpath='{.data.pass}' | base64 -d   # anyone can read it
```

**Remember:** A Secret only keeps data out of your YAML — lock it down with RBAC + etcd encryption.

`#CKA`  `#Kubernetes`  `#Secrets`  `#Security`

---

## Post 11 — port vs targetPort vs nodePort

Three ports on one Service and everyone mixes them up. **port** = the Service's own port (what clients hit inside the cluster). **targetPort** = the container port traffic is forwarded to. **nodePort** = the port opened on every node for external access (30000–32767).

```bash
ports:
  - port: 80          # the Service listens here
    targetPort: 8080  # forwards to container's 8080
    nodePort: 30080    # reachable on <nodeIP>:30080 (NodePort type)
```

**Remember:** Client → **port** → **targetPort** (container). **nodePort** is the external door.

`#CKA`  `#Kubernetes`  `#Services`  `#Networking`

---

## Post 12 — How a Service actually finds its pods

A Service is just a stable name and IP — the real work is done by **EndpointSlices** (the modern replacement for Endpoints). A controller watches pods matching the Service's selector and writes their IPs into slices. No ready pods matching the selector = empty endpoints = connection refused.

```bash
k get endpointslices -l kubernetes.io/service-name=web
k describe svc web        # check Selector vs pod labels
```

**Remember:** Service unreachable? Check EndpointSlices. Empty = selector mismatch or no ready pods.

`#CKA`  `#Kubernetes`  `#Networking`  `#Troubleshooting`

---

## Post 13 — Cluster DNS is predictable, so memorize it

Every Service gets a DNS name with a fixed shape. Same namespace? Just use the Service name. Cross-namespace? Add the namespace. The full form always works:

```bash
<service>.<namespace>.svc.cluster.local
# e.g. db.prod.svc.cluster.local
k run test --image=busybox --rm -it -- nslookup db.prod
```

**Remember:** `service.namespace.svc.cluster.local` — same-ns you can drop everything after the service name.

`#CKA`  `#Kubernetes`  `#DNS`  `#Networking`

---

## Post 14 — Default-deny is the NetworkPolicy foundation

Pods talk to everything by default. NetworkPolicies are additive allow-rules — but the powerful first step is a default-deny that closes the namespace, then you open only what's needed. An empty `podSelector` selects *all* pods.

```bash
kind: NetworkPolicy
spec:
  podSelector: {}          # all pods in the namespace
  policyTypes: ["Ingress"]  # no ingress rules = deny all ingress
```

**Remember:** Empty podSelector + a policyType with no rules = default-deny. Then layer allow-policies on top.

`#CKA`  `#Kubernetes`  `#NetworkPolicy`  `#Security`

---

## Post 15 — An Ingress without a controller does nothing

Creating an Ingress resource is only half the job — it's just rules. Something has to *implement* them: an Ingress controller (nginx, Traefik, etc.). No controller running = your Ingress sits there with no address. Meanwhile the newer **Gateway API** is where the ecosystem is heading, with richer routing and role separation.

```bash
k get pods -n ingress-nginx      # is a controller actually running?
k get ingress web                # ADDRESS empty? no controller.
```

**Remember:** Ingress = rules; the controller = the engine. Gateway API is the successor.

`#CKA`  `#Kubernetes`  `#Ingress`  `#Networking`

---

## Post 16 — PVC binding rules (and the Pending trap)

A PVC binds to a PV only if **capacity ≥ request**, **access modes match**, and **storageClassName matches**. Miss any one and the PVC sits Pending. And with a topology-aware StorageClass, `WaitForFirstConsumer` intentionally delays binding until a pod is scheduled — so Pending is expected until then.

```bash
k describe pvc data     # Events tell you exactly why it's Pending
```

**Remember:** PVC Pending? Compare size, accessModes, storageClassName. If mode is WaitForFirstConsumer, it binds only once a pod uses it.

`#CKA`  `#Kubernetes`  `#Storage`  `#Troubleshooting`

---

## Post 17 — The "Released" PV won't come back on its own

Delete a PVC and the PV's reclaim policy decides what happens. **Delete** wipes the backing storage. **Retain** keeps your data — but marks the PV `Released`, and a Released PV will *never* auto-bind to a new claim. You have to manually clear its `claimRef` to make it Available again.

```bash
k patch pv pv-data -p '{"spec":{"claimRef":null}}'   # Released -> Available
```

**Remember:** Retain protects data but strands the PV in Released — clear `spec.claimRef` to reuse it.

`#CKA`  `#Kubernetes`  `#Storage`  `#SRE`

---

## Post 18 — nodeSelector is a hard filter; affinity is a language

**nodeSelector** is the blunt tool: exact label match, all-or-nothing. **Node affinity** is the expressive version — it supports operators (In, NotIn, Exists), and both *required* (hard) and *preferred* (soft) rules. Use nodeSelector for simple pins, affinity when you need "prefer, but don't fail."

```bash
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - {key: disktype, operator: In, values: ["ssd"]}
```

**Remember:** nodeSelector = exact-match hard rule. Affinity = operators + hard/soft preferences.

`#CKA`  `#Kubernetes`  `#Scheduling`  `#DevOps`

---

## Post 19 — Taints repel, tolerations permit

A **taint** on a node says "keep pods away unless they can tolerate me." A **toleration** on a pod says "I'm okay with that taint." Key point: a toleration doesn't *attract* a pod to the node — it just removes the repulsion. Use affinity if you actually want to pull pods in.

```bash
k taint nodes node1 gpu=true:NoSchedule
# pod tolerates it:
tolerations: [{key: gpu, operator: Equal, value: "true", effect: NoSchedule}]
```

**Remember:** Taint = node repels; toleration = pod is allowed (not attracted). Effects: NoSchedule, PreferNoSchedule, NoExecute.

`#CKA`  `#Kubernetes`  `#Scheduling`  `#SRE`

---

## Post 20 — DaemonSets: one pod per node, automatically

Need an agent on every node — a log shipper, a CNI, a metrics collector? That's a DaemonSet. It runs exactly one pod per matching node and automatically schedules onto new nodes as they join the cluster. No replica count to manage.

```bash
k get ds -A          # DESIRED matches node count
k get pods -o wide   # one pod per node
```

**Remember:** DaemonSet = one pod per node, self-adjusting as nodes come and go. It tolerates control-plane taints if you tell it to.

`#CKA`  `#Kubernetes`  `#DaemonSet`  `#Workloads`

---

## Post 21 — Jobs run to completion; CronJobs run on a schedule

Deployments keep pods running forever. A **Job** is the opposite — it runs a task until it succeeds, then stops. A **CronJob** is just a Job factory on a cron schedule. Watch the `restartPolicy`: Jobs use `Never` or `OnFailure`, never `Always`.

```bash
k create job pi --image=perl -- perl -Mbignum -e 'print bpi(200)'
k create cronjob report --image=busybox --schedule="0 * * * *" -- date
```

**Remember:** Job = run once to completion. CronJob = Job on a schedule. restartPolicy must be Never/OnFailure.

`#CKA`  `#Kubernetes`  `#Jobs`  `#Workloads`

---

## Post 22 — Role vs ClusterRole, and the question that ends debates

RBAC has two axes. **Role** is namespaced; **ClusterRole** is cluster-wide (and covers cluster-scoped resources like nodes). A RoleBinding grants within a namespace; a ClusterRoleBinding grants everywhere. When in doubt about what someone can do, just ask the API.

```bash
k auth can-i create pods --as=jane -n dev
k auth can-i '*' '*' --as=system:serviceaccount:dev:builder
```

**Remember:** Role/RoleBinding = namespaced. ClusterRole/ClusterRoleBinding = cluster-wide. `auth can-i` settles every argument.

`#CKA`  `#Kubernetes`  `#RBAC`  `#Security`

---

## Post 23 — Every pod has an identity, whether you set it or not

Pods authenticate to the API server as a **ServiceAccount**. Don't specify one? You get the namespace's `default` SA. A projected token is mounted into the pod, and RBAC bindings on that SA decide what the pod's code can do. This is how you give a controller pod exactly the permissions it needs — no more.

```bash
k create sa builder
k set serviceaccount deploy/ci builder   # bind SA to workload
```

**Remember:** No SA specified = the default SA. Grant least privilege by binding a Role to a dedicated ServiceAccount.

`#CKA`  `#Kubernetes`  `#ServiceAccount`  `#Security`

---

## Post 24 — Static pods answer to the kubelet, not the scheduler

Drop a manifest in `/etc/kubernetes/manifests` and the kubelet runs it directly — no API server, no scheduler involved. That's how the control plane bootstraps itself (apiserver, etcd, controller-manager all run as static pods). You see a read-only mirror pod in the API, but you can't delete it with kubectl — remove the file.

```bash
ls /etc/kubernetes/manifests/     # kube-apiserver.yaml, etcd.yaml ...
# edit or move a file to change/stop a static pod
```

**Remember:** Static pods live in `/etc/kubernetes/manifests` and are managed by editing files, not kubectl.

`#CKA`  `#Kubernetes`  `#kubelet`  `#ControlPlane`

---

## Post 25 — The etcd snapshot task you *will* see

etcd holds all cluster state, and backing it up is a near-guaranteed CKA task. Memorize the flags — the three certs plus the endpoint. Save is easy; restore points to a fresh data-dir, then you repoint the etcd static pod at it.

```bash
ETCDCTL_API=3 etcdctl snapshot save /opt/snap.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

**Remember:** save needs endpoint + cacert + cert + key. Restore with `--data-dir` to a new path, then update the etcd manifest.

`#CKA`  `#Kubernetes`  `#etcd`  `#Backup`

---

## Post 26 — Upgrade order: control plane first, one minor at a time

kubeadm upgrades follow a strict order and you cannot skip minor versions. Do the first control-plane node with `kubeadm upgrade apply`, then other control-plane nodes with `upgrade node`, then workers. On each node, upgrade the kubeadm binary first, then kubelet + kubectl. Drain before, uncordon after.

```bash
kubeadm upgrade plan
kubeadm upgrade apply v1.30.2      # first control-plane node
k drain node --ignore-daemonsets   # before touching kubelet
```

**Remember:** Control plane → workers, one minor at a time. Never jump 1.29 → 1.31.

`#CKA`  `#Kubernetes`  `#kubeadm`  `#Upgrades`

---

## Post 27 — cordon, drain, uncordon: the maintenance dance

Three verbs for taking a node out of service safely. **cordon** marks it unschedulable (existing pods stay). **drain** cordons *and* evicts the pods, respecting PodDisruptionBudgets. **uncordon** puts it back in rotation. You almost always need `--ignore-daemonsets`, and often `--delete-emptydir-data`.

```bash
k drain node1 --ignore-daemonsets --delete-emptydir-data
# ... do maintenance ...
k uncordon node1
```

**Remember:** cordon = stop new pods; drain = cordon + evict; uncordon = back to work.

`#CKA`  `#Kubernetes`  `#NodeOps`  `#SRE`

---

## Post 28 — The 4-layer troubleshooting method

When something's broken, don't poke randomly — descend the stack in order. **App** (logs, describe, events) → **Control plane** (are apiserver/scheduler/controller-manager healthy?) → **Node** (kubelet, disk, Ready status) → **Network** (Service endpoints, DNS, NetworkPolicy). Most CKA failures resolve in the first two layers.

```bash
k get events -A --sort-by=.lastTimestamp   # start here, always
k get pods -n kube-system                    # control-plane health
k get nodes                                   # node Ready?
```

**Remember:** App → Control plane → Node → Network. Start with events, sorted by time.

`#CKA`  `#Kubernetes`  `#Troubleshooting`  `#SRE`

---

## Post 29 — When kubectl can't help you, drop to the node

If the API server or kubelet is down, kubectl is useless — so you go directly to the node. **crictl** talks to the container runtime (list/inspect/log containers). **journalctl -u kubelet** shows why the kubelet won't start. This is the exam's "the node is NotReady, fix it" scenario.

```bash
crictl ps -a                       # containers the runtime sees
journalctl -u kubelet -f           # why kubelet is unhappy
systemctl status kubelet
```

**Remember:** No kubectl? crictl for containers, `journalctl -u kubelet` for the kubelet's own logs.

`#CKA`  `#Kubernetes`  `#crictl`  `#Troubleshooting`

---

## Post 30 — Exam day: killer.sh, the clock, and the flag button

The two free **killer.sh** sessions are harder than the real exam on purpose — treat them as your calibration. On the day: skip anything that stalls you and **flag it** for later, watch the weight of each question, set your context with `kubectl config use-context` every single task, and verify your work. Time management beats knowing every answer.

```bash
k config use-context <ctx>   # FIRST thing, every question
```

**Remember:** Do killer.sh twice, flag hard questions, switch context first, and never let one task eat your clock.

`#CKA`  `#Kubernetes`  `#ExamPrep`  `#Certification`

---
