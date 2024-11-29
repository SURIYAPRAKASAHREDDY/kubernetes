# DaemonSets

 Daemonsets is help to create 1 pod in 1 running worker node. if any worker node failure there is no imapct on prod servers.

 if we have 6 pods in 3 Nodes ?
    daemonSet wont allow to create 6 pods in 3 nodes. it going to create each pod in each node. if we use nodeselector it will create according to node.

#   Summary of Approaches to Create 6 Pods
DaemonSet alone will create 3 pods (1 per node).
If you want 6 pods, use:
Multiple DaemonSets with node selectors.
A DaemonSet and a Deployment with replicas.

# daemonset.yaml
     helps to create daemonse, 
     commands:
       - kubectl explain daemonset
       - kubectl apply -f daemonset.yaml
       - kubectl get daemonset -n <name space>
       - kubectl describe daemonset/name of daemonset -n <name space>
       - kubectl get pods -n <name space>
       - kubectl delete daemonset/ name of deamonsets - n name space

# cron jobs

A CronJob in Kubernetes is similar to a cron job in Unix/Linux systems. It allows you to run jobs on a scheduled basis, and it follows a cron expression format. The cron syntax consists of five fields that represent:

* * * * *  <command to run>
| | | | |
| | | | +---- Day of the week (0 - 6) (Sunday=0)
| | | +------ Month (1 - 12)
| | +-------- Day of the month (1 - 31)
| +---------- Hour (0 - 23)
+------------ Minute (0 - 59)

# for every minute  | for every min on sunday  | for every month 1st sunday at 11:30 pm

*/1 * * * *         | */1 * * * 0              | 30 23 1-7 * 0 



kubectl get pods -n surya                              
NAME                        READY   STATUS      RESTARTS   AGE
my-cronjob-28880739-q4g67   0/1     Completed   0          2m15s
my-cronjob-28880740-cgv95   0/1     Completed   0          75s
my-cronjob-28880741-rzfsw   0/1     Completed   0          15s
surya-daemonset-nc5lh       1/1     Running     0          22m
kubernetes\daemonSet> kubectl  logs pod/my-cronjob-28880739-q4g67 -n surya
surya cronjob working
kubernetes\daemonSet> kubectl get jobs -n surya                              
NAME                  STATUS     COMPLETIONS   DURATION   AGE
my-cronjob-28880739   Complete   1/1           3s         2m33s
my-cronjob-28880740   Complete   1/1           2s         93s
my-cronjob-28880741   Complete   1/1           3s         33s
kubernetes\daemonSet> kubectl get cronjobs -n surya                          
NAME         SCHEDULE        TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
my-cronjob    */1 * * * *    <none>     False     0        41s             3m27s
kubernetes\daemonSet> 


kubectl delete cronjobs/my-cronjob -n surya
cronjob.batch "my-cronjob" deleted


kubectl delete daemonset/surya-daemonset -n surya 
daemonset.apps "surya-daemonset" deleted