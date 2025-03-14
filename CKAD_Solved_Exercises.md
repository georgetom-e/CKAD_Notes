Tips for the Exam: 

# ATTEMPT ALL EASY QUESTIONS FIRST, DON'T GET STUCK, RETURN LATER. 

# GET GOOD WITH YAML (practice indentation) 

# USE ALIASES (po, pvc, rs, np...) 

# USE IMPERATIVE COMMANDS. Unsure? use command --help | grep -i "goalKeyword"


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


LABELS, DEPLOYMENTS 

Use imperatve --labels to create pod with label 

Get pods with the app=grey label: k get pods --selector=app=grey 

IMPORTANT: use k label / k annotate to make changes in labels after object creation

Q) Rename a pod withe label app=blue to app=red 
   k label --overwrite pod podName app=blue 


Q) Add a new label tier=web to all pods having 'app=v2' or 'app=v1' labels
   k label po -l "app in (v1, v2)" tier=web 


Q) Remove the label app=blue from a pod p2
   k label po p2 app-

Q) Annotate a pod p1 with the annotation version=3.14
   k annotate pod p1 version=3.14

 Q)Now remove this annotation 
   k annotate pod p1 versio- 

POD PLACEMENT 

Q) Create a pod that will be deployed to a Node that has the label 'accelerator=nvidia-tesla-p100'

  1. label the node: k label node control-plane accelerator=nvidia
  2. create pod with node selector to that label: k run pod --image-nginx > ans.yaml, vi ans.yaml to add nodeSelector: acclerator=nvidia

Q) Taint a node with key tier and value frontend with the effect NoSchedule. Then, create a pod that tolerates this taint.

  k taint node controlplane tier=frontend:NoSchedule 
  k run pod p1 --image=nginx --dry-run=client -o yaml > ans.yaml
  vi ans.yam to include tolerations of tier equal frontend

Q) Use Node affinity to achieve the same as above 

    vi ans.yaml, modify to include: 

    spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: tier
            operator: In
            values:
            - frontboy

DEPLOYMENT STRATEGIES 


Blue-Green: two versions of the app (blue, green) label blue pods v1, green pods v2, create a svc to select pods with label v1, then change svc's selector to match green pods v2.



Canary: traffic to both deployment versions-common label; trick is to have fewer pods of the newer version to reduce traffic on the new version. 

Q) change the image of a deployment; this triggers a rollout.
   
   k set image deployment depName containerName=newImage 

Q) check the rollout status, history, pause, resume undo the rollout

   k rollout status/history/pause/resume/undo deployment depName

Q) rollback to revision 2 

  k rollout undo deployment depName --to-revision=2

Q) Autoscale the deployment, pods between 5 and 10, targeting CPU utilization at 80%

  k autoscale deployment depName --min=5 --max=10 --cpu-percent=80 (THIS CREATES A HPA, k get hpa)

Q) Implement canary deployment by running two instances of nginx marked as version=v1 and version=v2 so that the load is balanced at 75%-25% ratio

   (75-25 => 3: 1 replicas)

   Crude High Level: 

   1. create deployment nginx-v1 (replicas=3), nginx-v2 (replicas=1); label both the same (strat: canary-ng)
   2. create a svc that selects both versions via the canargy-ng label

   Actual Deployment Strategy: 

   1. Create deployment nginx-v1 replica=3 label canary-ng, for testing purposes let the container execute echo hello. 
   2. Create a svc that connects to the canary-ng label; test connectivity by performing a wget from a test busybox pod to the svc. 
   3. Create deployment nginx-v2 replicas=1 with label canary-ng; test connectivity again too see traffic shared between both versions in the ratio 3:1 

    k create deployment nginx-1 --image=nginx --replicas=3 -- /bin/sh -c "echo from v1" --port 80 
    k create deployment nginx-2 --image=nginx --replicas=1 -- /bin/sh -c "echo from v2" --port 80 
    k label deployments.apps nginx-1 strat=canary
    k label deployments.apps nginx-2 strat=canary
    k create svc clusterip canary-svc --tcp 80:80 --> edit svc to include selector labels for start=canary

    test traffic: 
   
IMPORTANT: kubectl run test-pod --rm -it --image=busybox -- /bin/sh  -> to get a network test terminal 
           while true; do wget -qO- canary-svc:80; echo; sleep 1; done  -> to test network within that tets container. 

JOBS AND CRONJOBS 

Q) Create a job named pi with image perl:5.34 that runs the command with arguments "perl -Mbignum=bpi -wle 'print bpi(2000)'"
   k create job pi --image=perl:5.34 -- perl -Mbignum=bpi -wle 'print bpi(2000)'

Q) Create a job with the image busybox that executes the command 'echo hello;sleep 30;echo world'
   k create job testjob --image=busybox -- /bin/sh -c "echo hello; sleep 30; echo world"

Q) Create a job but ensure that it will be automatically terminated by kubernetes if it takes more than 30 seconds to execute
  IMPORTANT: .spec.activeDeadlineSeconds: x 
  IMPORTANT: ONLY FIVE STARS IN SCHEDULE, every minute: */1 * * * *; every hour: * */1 * * * 
   k create jonb uselessjob --image=busybox -- /bin/sh -c "echo hi; sleep 32; echo world" --dry-run=client -o yaml > ans.yaml 
   ans.yaml -> activeDeadlineSeconds: 30 

Q) Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it takes more than 17 seconds to start execution after its scheduled time (i.e. the job missed its scheduled time).
   k create cronjob clock --image=busybox --schedule='*/1 * * *' -- /bin/sh -c "date" --dry-run=client -o yaml > ans.yaml 
   
Q) Create a job from an existing cron job 
   k create job whatever --from=cronjob/clock 


SERVICES, INGRESS 

Q) Create a nginx pod, expose it through a clusterip service, then access the nginx pod through the service; 

    1. k run nginx --image=nginx 
    2. k expose pod nginx --port=80 (creates a ClusterIp by default with same name as pod, port 80 is the container's port; service port also 80)
   3. k get endpoints nginx and double test as below: 
IMPORTANT: test service endpoints --> k run test --image=busybox --rm -it -- /bin/sh --> open test pod's terminal 
            then in terminal: while true; do wget -qO- clusterIP:port ;echo ; sleep 5; done

Q) Convert this clusterIP service into a NodePort type --> change def file tyep: 


IMPORTANT: it's always faster to expose an object rather than creating a service and then creating a selector-label situation. 
