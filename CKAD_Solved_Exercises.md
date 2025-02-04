
A. CORE CONCEPTS 


1. Create a pod declaratively, using YAML (DRY RUN) 

k run mypod --image=nginx --dry-run=client -o yaml > 2.yaml 
modify 2.yaml as as needed, k apply -f 2.yaml 


2. To run a command in a sneaky pod that completes and exits before you can k exec. Pass the command during pod creation

kubectl run busybox --image=busybox --command --restart=Never -- env

k logs busybox to see the o/p

3. All pods in all namespaces: k get pods --all-namespaces 

4. k explain <object> to get object help 


5. how to get pod's IP? k get pod -o wide 
   how to test this IP: k run busybox --image=busybox --command wget IP; check busybox logs, you'll see it successfully pulled data from that IP


6. how to view logs from a previously crashed and restarted container?
    k logs podName -p (previous) 


7. create a busybox pod that echoes "hello world" and exits 

   k run busyboxHello --image=busybox -it -- echo "hello world"

the same, but auto delete the pod when done: --rm flag after -it 


8. create a busybox pod with env variable var1=val1, enter the container and verify this

k run busybox --image=busybox --env=var1=val1
k exec busybox -- env 


MULTI-CONTAINER PODS 

Create a Pod with two containers, both with image busybox and command "echo hello; sleep 3600". Connect to the second container and run 'ls'

KEY: when you have a complex shell command: pass -- /bin/sh -c "list of commands" during creation 
Solution: 


 k run test --image=busybox --dry-run=client -o yaml -- /bin/sh -c "echo 'hello world'; sleep 1000" > ans.yaml 
 vi ans.yaml 
 k apply -f ans.yaml 

you can go into any of the containers to perform any command now, since you've added a sleep 1000. 

k exec test1/2 -- ls 

VOLUME MOUNT PENDING QUESTION 


HELM 

How to create a helm chart? 

helm create test-chart 


How to run a helm chart? 

helm install release-name chart-name 


How do you perform repo operations? 

helm repo add/list/remove/update 

helm repo add yourRepoName repoURL


How do you simply pull a chart, not install it?

helm pull --untar chartName 


How do you show the values of a helm chart? 

helm show values chartName 


IMPORTANT: How do I change a particular value in the values.yaml and then install teh chart? 
Q) set the replicas=5 and install a bitnami/node chart 

a) helm show values bitnami/node | grep -i replica  --> to see whoch value has to be set 
b) helm intsall bitnami/node --set replicaCount=5 


OBSERVABILITY 


Create a busybox pod that runs i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done. Check its logs

NOTE: enclose the long list of commands in '' since " " is used within the commands, and this can confuse K8. 

k run shellpod --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done' > ans.yaml 

k logs shellpod 

Create a busybox pod that runs 'notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod forcefully with a 0 grace period

k delete pod errorpod --force --grace-period=0 


For metrics; k top node/pod, metrics server must be installed.


Create an nginx pod with a liveness probe that just runs the command 'ls'. Save its YAML in pod.yaml. Run it, check its probe status, delete it.Modify the pod.yaml file so that liveness probe starts kicking in after 5 seconds whereas the interval between probes would be 5 seconds. Run it, check the probe, delete it.

k run test --image=nginx --dry-run=client -o yaml > ans.yaml 
vi ans.yaml (include liveness probe, initialDelaySeconds, periodSeconds from K8 docs) 
k apply -f ans.yaml 



Create an nginx pod with a liveness probe that simply runs the command ls (dry run and follow documentation) 


IMPORTANT: Lots of pods are running in qa,alan,test,production namespaces. All of these pods are configured with liveness probe. Please list all pods whose liveness probe are failed in the format of <namespace>/<pod name> per line.


HINT: use k get events, which is a master log of events across namespaces. Then filter your result using the JSON jq module in shell. 

Crude way: k get events | grep -i "failed..." | awk '{print $3}' or whatever column to find the pod name and namespace. 

Better way: kubectl get events -o json | jq -r '.items[] | select(.message | contains("Liveness probe failed")).involvedObject | .namespace + "/" + .name'



