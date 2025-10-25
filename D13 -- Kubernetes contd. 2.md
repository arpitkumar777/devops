#### D13 \& D14 (08 \& 09 October 2025) -- Kubernetes contd.



\- Create the worker script into a file

vim name.sh

chmod +x name.sh -- giving the executive permissions for the same

./ name.sh -- This starts the script and generates the entire configuration for the worker node automatically.





#### Lets work with Pods

-- In root/ master server:

\# mkdir /k8s-code

\# cd -"-

\# vim my-first-pod.yaml



apiVersion: v1

kind: Pod

metadata:

  name: devops

spec:

  containers:

  - name: web-app

    image: nginx:1.14

    ports:

    - containerPort: 80



\# kubectl apply -f my-first-pod.yaml

\# kubectl get pod

\# kubectl describe pod devops

\# cp my-first-pod.yaml second-pod.yaml

\# kubectl get pod

\# kubectl get ns

\# kubectl apply -f second-pod.yaml (container name can be same but not the pod name(ie., metadata), change the name: prod-pod)

\# kubectl describe pod prod-pod | grep -i controlled (to check for the status of controller - recreator in case of failure)

\# kubectl describe pod prod-pod



---

( When creating a Pod, it is of the best interest to provide the specs before launched ie., QoS classes: Best Effort, burstable and guaranteed)

Best Effort:

* Low Priority Pod
* When no specs given, gives random resources as per its minimal guarantee



Burstable:

* Pod has request or limit set
* Limits higher than request
* Minimum resource guaranteed
* Can consume upto limit



Guaranteed:

* Limits and requests are equal.
* Highest Priority
* Guaranteed not to be killed before "Best effort" and "Burstable" pods



---



-- Burstable Qos

\# vim my-first-pod.yaml



apiVersion: v1

kind: Pod

metadata:

  name: test-pod

spec:

  containers:

  - name: web-app

    image: nginx:1.14

    ports:

    - containerPort: 80

    resources:

      limits:

        memory: "200Mi"

      requests:

        memory: "100Mi"





\# kubectl apply -f my-first-pod.yaml

\# vim cpu\_burstable.yaml

apiVersion: v1

kind: Pod

metadata:

 name: cpu-burst

spec:

 containers:

 - name: dev-env

   image: vish/stress

   resources:

     limits:

 	cpu: "0.4"

     request:

 	cpu: "0.2"



\# kubectl apply -f cpu\_burstable.yaml

\# kubectl get pod

\# kubectl describe pod cpu-burst





--- Guaranteed QoS: (Make sure that both the RAM as well CPU resources are given, else considers it as **BURSTABLE** QoS)

\# vim guaranteed-pod.yaml



apiVersion: v1

kind: Pod

metadata:

 name: guaranteed-res

spec:

 containers:

 - name: guaranteed-container

   image: httpd

   resources:

    requests:

      memory: "100Mi"

      cpu: "0.4"

    limits:

      memory: "100Mi"

      cpu: "0.4"





\# kubectl apply -f guranteed-pod.yaml

\# kubectl describe guranteed-res



---



#### **ReplicaSet**



\# kubectl api-resources --- To check the versions and what the dependencies of Kubernetes env would do.



\# vim devops-replica.yaml

apiVersion: apps/v1

kind: ReplicaSet

metadata:

 name: devops-rs

 labels:

   app: devops-rs (can be anything)

   tier: prod

spec:

 replicas: 3

 selector:

  matchLabels:

    tier: prod

 template:

   metadata:

     labels:

       tier: prod

   spec:

    containers:

    - name: web-apache

      image: nginx (can be anything)

 

\# apply the pod

\# kubectl get rs

\# kubectl describe rs devops-rs

\# describe one of the replicas

\# kubectl delete pod --replica--



---



##### **Pod Auto Scaling:**

\# vim devops-replica.yaml

Change the "replicas" to 8

\# watch kubectl get pod (Freeze the process)



duplicate the terminal for the same session

# kubectl apply -f devops-replica.yaml

-- adds 5 new replicas of the same

\# change the replicas to 4

-- removes the replicas as per the time frame



###### **Vertical Scaling**

We can do this scaling without changing it in the yaml file:

\# kubectl scale replicaset devops-replica --replicas=10



###### **Horizontal Auto Scaling**

\# kubectl api-resource

  find horizontalpodscaler - APIVersion

\# vim my-hpa.yaml (Find the reference HPA code from the official documentation)

\# kubectl attach my-hpa.yaml

\# kubectl get hpa



---

###### ***Isolating Pods from replicaset:***



***# kubectl label pod devops-rs(pod-name) app=debug --overwrite***

***# kubectl describe pod devops-rs***





###### **NOTE: If replicaset gets crashed or deleted, then the Pods created will also be deleted -- Thereby failing my applications.**

**Use "Deployment" on top of replicaset that maintains the redundancy to replicaset.**

**Create a manifest file for that.**



**------------------------------------------------------------------------------------------------------------------------------------------------------------**



###### **Deployment:**



**# vim deployment-code.yaml**

apiVersion: apps/v1

kind: Deployment

metadata:

 name: nginx-deployment

 labels:

   app: web(can be anything)

spec:

 replicas: 1

 selector:

  matchLabels:

    app: web

 template:

   metadata:

     labels:

       app: web

   spec:

    containers:

    - name: web

      image: nginx:1.14 (can be anything)

      ports:

      - containerPort: 80



\# kubectl apply -f deployment-code.yaml

\# kubectl get deploy

**# kubectl get rs**

**# kubectl get pod (Shows 1 Pod)**



###### **HPA: (Get the code from the official documentation wrt Deployment HPA)**



**# vim hpa.yaml**

apiVersion: autoscaling/v2

kind: HorizontalPodAutoscaler

metadata:

 name: my-app-hpa

spec:

 scaleTargetRef:

   apiVersion: apps/v1

   kind: Deployment

   name: nginx-deployment ***(Deployment name)***

 minReplicas: 4

 maxReplicas: 10

 metrics:

   - type: Resource

     resource:

       name: cpu

       target:

         type: Utilization

         averageUtilization: 50



\# kubectl apply -f hpa.yaml

\# kubectl get hpa

\# kubectl get pod (will show 3 more pods to the created deployed pods)



---



###### **Updating a deployment:**



Declarative -- Using the file

Imperative -- Single Command



\# kubectl describe deploy nginx-deployment

\# kubectl get pod (4 Pods)

\# vim deployment-code.yaml

change the replicas to 8

\# kubectl apply -f deployment-code.yaml

\# kubectl get pod



 ----------------------------------------------------------------------------------

| **Process Flow:                                                                    |**

| user -> service -> deployment -> replicaset -> pods -> containers -> application |

 ----------------------------------------------------------------------------------



To update image version:

\# kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1   (nginx= refers to the container name. In my case, it is "app" as referred to in the deployment yaml manifestation file)

On update, removes the older replicaset, shifts to the new replicaset by storing the previous version of the application to rollback as well.



\# kubectl edit deploy nginx-deployment (The main config file of the deployment)

There exists a field called strategy: when there exists a load, changes the update as in the % value specified

maxSurge: The maxSurge property controls the maximum number of additional pods that can be created during a rolling update.

maxUnavailable: The maxUnavailable property determines the maximum number or percentage of pods that can be unavailable during a rolling update.



Note: The failed status will not be shown in a previous image when update to a new image/ RS. When checked before updating, we can look at the status.



 rollout \& rollout history



\# to check the rollout history:

kubectl rollout history deployment/nginx-deployment



\# To move to a specific revison number: kubectl rollout undo deployment/nginx-deployment --to-revision=2



undo (Move one step behind)

\# kubectl rollout undo deployment/nginx-deployment



 vertical scaling:

\# kubectl scale deployment/nginx-deployment --replicas=10



 pause:

\# kubectl rollout pause deployment/nginx-deployment

and update as many as changes to the deploy, check for histroy, no new entries will be made.

resume:

\# kubectl rollout resume deployment/nginx-deployment - You can see the updates being made when checked for history

 



resource utilization:





---



Namespaces:
Split the existing cluster base into smaller bases to different uses which makes sure that it doesn't seem like a different cluster to them.



Every pod has a name and an IP.

C-group: responsible to provide the necessary hardware resources

namespace: responsible to provide the software support



 

\# vim my-name-space.yaml

apiVersion: v1

kind: Namespace

metadata:

 name: prod



\# apply

\# kubectl get ns

\# kubectl create ns dev-env



\# vim my-name-space.yaml

apiVersion: v1

kind: Namespace

metadata:

 name: web-app

 namespace: web1

spec:

 containers:

 - name: apache

   image: nginx:1.14



\# kubectyl apply -f my-name-space.yaml --namespace=pod

\# kubectl get pod -n prod





 

