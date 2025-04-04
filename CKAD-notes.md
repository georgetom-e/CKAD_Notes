Create separate ingress resources for separate namespaces.

in kubeapi.yaml add, --runtime-config=versionneeded just like admission controller changes

k api-resources --> to see all API versions and groups.

k explain <object> --> to see detailed info about a object.

k convert -f oldFile --output-version <newApi>

API Deprecations: 

  

Rule 1:  API elements may only be removed by incrementing the version of the group. That is, the deprecated API element will remain in its original version; it will be excluded only in newer version.  
  
Rule 2: API objects must survive a roundtrip without info loss; object v1 --> v2 --> v1; then v1 must be the same as the original v1.

to enable a certain api version: --runtime-config=batch/v2alpha in the kube-apiserver.yaml

Mutating Admission Controller actually goes ahead and fixes, mutates, the configuration issue. 

The Mutating AC is always run before non-mutating ACs.  

Custom Mutating ACs can be created --> MutatingAdmissionWebhook & ValidatinAdmissionWebhook

  

You deploy an AdmissionWebhook server with your 'allowed-logic' and a MAW AC running on K8; these two communicate and decide if operations are allowed.

admission controllers are responsible for allowed actions on the cluster, like image policy, label enforcement.

  

k exec kube-api-server-controleplane -n kube-system -- kube-apiserver -h | grep -i enable admission-plugins to check what admission controllers are enabled.  
  
**/etc/kubernetes/manifests/kube-apiserver.yaml** make admission control changes here: to enable or disable a admission controller plugin  

k exec -it kube-api-pod -n kube-system | kube-api-server -h | grep -i admission to see the list of default enabled plugins

Objects are either namespaced or cluster scoped (like nodes, PV, namespaces themselves).

To see these objects: k api-resources --namespaced=true/false

  

clusterroles & clusterrolebindings are just like roles & rolebindings, except that these apply to the whole cluster. They are used for objects that are cluster scoped (like PVs) but can also be used for namespaced objects, in case the role permission should apply at the cluster level.

RBAC points: 

  

*   Roles and Role Bindings **apply only that namespace**
    
*   Roles --> permission template; Role Bindings --> user-role assignment
    
*   k auth can-i get pods --> checks if current user can perform an action
    
*   k auth get pods --as otherUser or k auth
    

**resourceName** field in roles can define further filtering of type of resources to be granted in roles.

\--authorization-mode=AlwaysAllow (allows everything) is the default auth mode set on the kube api server.

  

RBAC Role Based Access Control: Create roles with pre-defined permissions and assign users to those roles.

  

1\. Define a Role

2\. Create a Role Binding, assigning a user to that Role

  

k get roles

k get rolebindings

All resources in K8 are categorised in api groups: named apis (newer, for the future) old apis.

  

api -> apigroup (/apps, /networking.k8.io...) -> resources (objects: deployments...) --> verbs (get, describe, delete...) 

the kubeconfig file contains user info and is located at the .kube directory of the user's home dir.

$HOME/.kube/config

kubectl picks this file from the .kube location as default.

  

the kubeconfig file has Clusters, Contexts, Users; context is simply the marraige of clusters and users, giving use what us what user had access to what cluster : cluster google + user dev=dev@google

  

Changes made to the kubeconfig file are auto-applied

  

current-context is the default context picked up.

  

**k config view --> to see the config file**

**k config use-context contextName --> to change current context**

  

IMP: **k config kubeconfig=/root/kubeconfigfile use-context research** use the kubeconfig= command when you want to use a file that is located outside the default .kube dir

**every rule should be written in an independent -to or -from and the allowed ports should be specified within that to / from**

  

  egress:

  - to:

    - podSelector:

        matchLabels:

          **name: mysql**

    ports:

    - protocol: TCP

      port: 3306

  

  - to:

    - podSelector:

        matchLabels:

          **name: payroll**

    ports:

    - protocol: TCP

      port: 8080

  

  - ports:

    - port: 53

      protocol: UDP

    - port: 53

      protocol: TCP

  

By default, all objects can talk to each other in a K8 network.

  

Ingress: like Import, traffic into an object

Egress: like Export, traffic out of an object

  

A network policy is a K8 object that can restrict ingress/egress on a specific pod using labels and selectors.  
  
The moment you apply a network policy type; all traffic of that type (ingress/egress) is blocked except what you mention in the np file.

  

How does the np know what pod to apply the np on? 

np file -> spec.podSelector.matchLabels.key=value that matches the desired pod

  

Ingress type np --> specify the allowed from: connections --> podSelector, namespace Selector, select by IP 

  

IMPORTANT! The list of rules under from/to are OR conditions if they are independent elements of the from/to list, and are AND conditions if part of the same rule list element.

  

k get ingress to get ingress rule objects

  

You can imperatively create an ingress rule like this: 

  

**k create ingress ingressName  --rule="www.google.com/maps=maps-service:80" --rule="www.google.com/docs=docs-service:82"**

  

use the rewrite target annotation in the ingress resource file to prevent incorrect redirections: 

  

Ingress is a built-in Load Balancer that allows you to expose a single application URL to the end-user, while configuring the route logic internally, depending on the requested service.

  

Implement Ingress in 2 steps: 

  

1\. Deploy Ingress Object --> Ingress Controller **(not deployed on cluster by default!)** 

2\. Configure Routing Logic --> Ingress Resources

  

You have rules for each domain name, and routing logics for various paths within that domain.

  

one rule for each URL, many paths under that rule for the routing logic

  

spec.rules:

  

\- http: (rule 1 for the first URL) --> paths --> paths in array --> under each path you have the target service.

  

you can define a default page when a user hits an undefined path.

**PV -> PVC -> mounted on Pod as a volume**

  

_The name of the pod volume must be the same as the PVC's volume name._

  

A volume object is configured on each pod.

You mention the mountPath: path on pod where the data will be mounted

You mention the hostPath: path on node where the data will be mounted.

An administrator creates PVs, while the user creates PVCs.

The claims are mapped and bound to a single PV.

For a PVC to bind to a PV, parameters like storage must make sense, **access Modes must match**

A PV is a centralised storage object that any pod with a valid PV claim can use.

The host path on a persistent volume indicates the storage path on the local node's directory (not to be used for production) 

In a production environment, you will replace the host path with a 3rd party cloud storage solution

**Use k get endpoint svcName to check where the service is routing traffic to.**

ClusterIP is the default service type for K8 internal communication

you specify the port, targetport

Once done, any pod in the same cluster can access the service by using the service name or service's IP.

NodePort: NodeIP:nodePort

  

While creating a nodePort, the only mandatory field is the port (of the service), the nodePort is auto assigned between 30, 000 and 32,767, and targetPort is autoset to same as port.

  

All from the POV of the service: 

  

port-- port on service (you can access the service from here, if you want ) 

targetPort--pod's target port

nodePort-- port on node.  (you will use this port to interact with the destination pod) 

  

  

selector: is used to map the service to the required pod's label.

Jobs

Completions is the Job term for pod replicas

Sequential task execution of pods is default behaviour.

Parallelism property will allow that many pods to run parallelly.

  

completions: 3 => 3 pods doing the task sequentially.

  

completions: 3

parallelism: 3 => 3 pods doing the task parallelly

  

Cronjobs

\* \* \* \* \* command to be executed

minute, hour, day of the month, month, day of the week (0Sunday-6Saturday) 7 also Sunday in some systems.

  

Look at the docs for the .yaml file.

  

  

STRATEGIES: 

  

Have to be implemented using service-label trickery.

  

Blue-Green: two versions of the app (blue, green) label blue pods v1, green pods v2, create a svc to select pods with label v1, then change svc's selector to match green pods v2. **delete old version blue.**

  

Canary: traffic to both versions; common label to both versions, trick is to have fewer pods of the newer version to reduce traffic into the new version. **once new version stability is established, scale down replicas in accordance with load balancing requirements.**

  

**The end-goal of these strategies is to switch to a newer version; once stability of that version is established, delete /scale down the older version' deployment.**

Any deployment created will create a rollout, which culminates in a revision number.

  

**k rollout status deployment deploymentName**  -> tp see the rollout status

**k rollout history deployment deploymentName** -> to see the revison history  

**How do I trigger a rollout?  
Simply modify the deployment file and apply.**

OR **k set image deployment deploymentName containerName=newimage (Note: this won't change your def file!)** 

  

DEPLOYMENT STRATEGIES: 

  
1\. Recreate Strategy: everything taken down, then brought up (bad idea) 

2\. Rolling Strategy: updated one pod at a time (default, good idea) 

  

See this in action under the events of a deployment.  
  
A deployment first creates a replicaSet, which in turn creates the required pods.

To undo a deployment: **k rollout undo deploymentName**

1\. ATTEMPT ALL EASY QUESTIONS FIRST, DON'T GET STUCK, MOVE ON.

IF YOU HAVE TIME, RETURN TO THE TOUGH QUESTIONS.  
  
2\. GET GOOD WITH YAML (practice indentation) 

3\. USE ALIASES (po, pvc, rs, np...) 

set labels on the yaml file, under metadata.

select via yaml file: 

selector:

  matchLabels: 

    app: blue

  

To selet via kubectl: 

k get pods --selector app=blue

  

extra version metadata may be found under annotations, under metadata.

BIG COPY CONCEPT FOR EXAM!

  

**While pasting yaml from the official documentation, you can sync-shift several lines by a tab by visually selecting required lines and then SHFT+.  (> key)** 

IMPORTANT concept

  

We need to combine taints-toleration and node-affinity/node-selector to guarantee that specific pods are scheduled on their specific nodes (achieved by node-selector, affinity) AND that no other pods are scheduled on those nodes (achieved by taints-toleration) 

To assign pods to specific nodes: 1. Node Selector 2. Node Affinity

  

1\. Node Selector

a) label node with a label: k label nodes node01 size=large

b) set nodeSelector on pod def: nodeSelector: size: large  
  
Limitation: single labels and match, no conditional expressions.

  

2\. Node Affinity  

refer K8 documentation, idea is that you have conditions and operators, longer syntax, careful.

  

Can be complex.

  

**Taint & Tolerations are about rejecting pods;** they do not guarantee that certain pods will always be scehduled on certain nodes. **They only guarantee that certain pods WILL NOT be scheduled on certain nodes.  Think of the taint applied on the master node /control plane that prevents app pods from being scheduled on it.**  

  

  
Taint- bug spray on a node

  

k taint nodes nodeName key=value:taint-effect ( **what to do if a pod without this key=value**  encounters your node --NoSchedule, PreferNoSchedule, NoExecute)

  

example: k taint node node01 app=blue: NoSchedule

  

untaint: same command as taint, just append a '-' at the end.

  

Toleration- bug spray immunity on a pod (applied in pod-def.yaml) 

apply the same what was specified in the taint condition; this will make the pod immune to the taint.

  

**Important to have values in quotes**

tolerations:

\-key: "app"

operator: "Equals"

value: "blue"

effect: "NoSchedule"

resource requests --> min guaranteed resources for a pod

resource limits --> max allowed

  

minimum cpu = 1 millicore (0.01 core) 

Gi, Mi, Ki are more than G, M, B.

  

If a container /pod tries and exceeds its cpu limits, the pod is throttled, however, if a pod exceeds the memory limit, the pod is restarted with an OOM message.

  

For CPU: Not setting limits (with requests set) may be beneficial for cases when you may need extra CPU.

  

**You can default limit resource consumption for new pods without limits, by creating a limitRange.yaml for CPU and Memory  
  
Resource Quota**

  

Limit resource usage on the namespace level, for everything on that namespace.

Helm chart = Template.yaml + Value.yaml + Chart.yaml (metadata) 

  

1\. Add a repo locally: helm repo add bitnami bitnamiURL  (helm repo ls to see all added repos) 

2\. To search helm repos: helm search repo chartName

3\. Install a helm chart: helm install \[release-name\] \[chart-name\] (uninstall to remove)

4\. List installed helm releases: helm list

5\. Only download, don't install: helm pull --untar chart-name

  

Every installation of a helm chart is called a release, each release has a version.

  

  

K8 doesn't recognise the relation between objects, or that all these objects function together as an app; it doesn't see the big picture. Helm does.

  

Helm simplifies K8 object management. You can change values in manifest files, change versions--like a git history, uninstall objects...

  

Helm is a package manager for K8.

  

Metrics server retrieves and aggregates cluster info. Note that this data is not stored; so there's no scope of historical data.

  

cAdviser in the kubelet on worker nodes exposes metrics to the kubeAPI server.

  

How do I enable metrics server? 

1\. git clone (metrics-server git repo)

2\. k create -f deploy/1.8+/

  

  

See the collected metrics:

k top node

k top pod

k logs -f podName containerName (in case you have more than one container) 

ReadinessProbe: tests if the app running on the pod is truly ready; use a HTTP API call to check ready status; define in pod.yaml

This prevents premature user-load on a new pod replica.

  

LivenessProbe: tests if the app is truly alive and healthy; K8 needs to know if containers are functioning properly; there needs to be an evaluate-condition for pod restarts.

Pod Status: 

1\. Pending -- Scheduler still doesn't know where to put the pod.

2\. Container Creating

3\. Running

  

Pod Conditions (T/F) 

Scheduled

Initialized

ContainersReady

Ready

If an init container fails, the pod will restart until the init container exits successfully, else the pod may fall into crash loop back off.

sidecar container - logging agent container (filebeat) alongside the main container.

adaptor container - converts current log format to a common format

ambassador container - handles log forwarding logic (some logs to a certain db, others to another db) 

  

Whatever the pattern, adding a new container is simply adding another element under the container array.

Service account is needed for 3rd-party apps accessing the cluster via APIs. Jenkins trying to deploy objects on a cluster.

  

k create sa saName

  

1\. Service account created

2\. create a token for this sa: **k create token  saName**

3\. K8 creates a secret (same name as token) object to store this token

  

_What if the application that needs authentication is hosted on the cluster itself?_ 

Then we can mount the service account token secret as a volume onto the 3rd party pod.

  

Every ns has its own service account. When a pod is created, this default token secret is mounted onto the pod.  But this token only gives you very basic permissions.

  

You can manually mount your new service account (but you must create a new pod for this; or modify this in the pod's parent deployment.)

  

  

If you add secrets as a volume, each secret is mounted as a file.

  

Secrets are only encoded, not encrypted. Secrets are not encrypted by default in the etcd; you have to enable encrypting secrets at rest. Anyone who can deploy a pod or deployment can also view secrets. Consider using a third-party secret management tool like Hashicorp Vault.

  

echo -n "textToBeEncoded" | base64 will encode your plaintext into base64, which is what secrets are encoded in.

  

echo -n "textToBeEncoded" | base64 --decode

  

  

k create secret **generic** --from-literal=key=value ...

  

k get secret secretName -o yaml to see the encoded values.

  

1\. Create Config Map

2\. Inject env variables into pod.

  

You can (1) inject the entire Config Map into a pod, or just one env variable of the Config Map (2), or add the entire config map as a volume on the pod (3).

  

1.  envFrom: 

       -configMapRef:

          name:configmap-name

  

2.

\- env:

    - name: APP\_COLOR

      valueFrom:

        configMapKeyRef:

          name: webapp-config-map

          key: APP\_COLOR

  

  

3\. volumes:

       name: configmap-name

Imperative commands: **no yaml creation,**

  

**k create configmap --from-literal=app-color=blue --from-literal=app-size=medium ...**

  

**k create configmap --from-file=filepath**

  

Declarative commmands: create yaml, then **k apply -f fileName**

Hardcode an environment variable: 

  

env:

  -name: APP\_COLOR

    value: pink

  

Or, pass it through a ConfigMap or Secret:

IMPORTANT 

  

In the pod def file: 

  

spec: 

command: \["sleep"\] --> override the ENTRYPOINT command of the docker file

args: \["10"\] --> overrides the CMD (arg) of the Dockerfile

ENTRYPOINT command will only accept a parameter passed. Nothing else.

CMD command will be overridden by the command / parameter passed. 

  

To have a default parameter, yet allow for a parameter to be passed and accepted: 

  

ENTRYPOINT \["sleep"\]

CMD \["5"\]

  

This will allow default sleep 5 and accept any other sleep parameter.

  

There is an exceptional force override of entrypoint command: docker run --entrypoint newcommand imageName 

docker build -t imageName:tag (tag-optional) . ( . to signify the Dockerfile is in the pwd) 

docker push imageName

Every Docker image is based off another image => FROM is always the first instruction in a Docker file.

Each instruction in a Docker file creates a layer in the final image. The first base image layer is the largest, the last layer is the smallest (memory).

**docker history imageName** to see the various layer info

In case you rebuild an image (failure in a step or a new step has to be added), previous layers need not be rebuilt, as they're stored in cache.  

Namespaces

  

1) Default (working default)

2) kube-public (resources that need to be available for all users)

3) kube-system (protected resources required for K8 functioning)

  

To change the current working namespace:

1.  kubectl config set\-context \--current \--namespace\=my\-namespace

  

**k get all --all-namespaces** to see all created objects in all namespaces

  

A resource quota can be used to limit and guarantee resources for namespaces.

A deployment manifest need only specify the pod details under spec; a replica set will be auto created.

  

Common errors in manifest files include typos, incorrect cases in apiVersion or kind

  

**Forgot a command's options? Use k <command> --help**

  

  

Don't remember kind / version for an object? Want to know more about the object?

  

Use: **k explain <object>** (pod, replicaset...)

Replication Controller ensures

1\. The specified number of pods are always running (replicas)

2\. Load Balancing: the RC has control over pod replicas whatever node they're on, multiple nodes (in case we have those many pods), so it can decide to scale up or down across nodes depending on the load.

ReplicaSet is the new object.