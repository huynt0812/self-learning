### 1. Manual scheduling
- When you define a pod-definition.yaml, in filed spec you define `nodeName`
- The K8s go to find the right node to run pod. 
- Once identified, it schedules the pod on the node by setting the node name property to the name of the node by creating a binding object. 


- So if there is no scheduler to minitor and schedule nodes, what happens, the pods continue to be in a pending state. You can manually assign pods to nodes yourself. 
- Another way to assign a node to an existing pod is to create a binding object adn send a post request to the pod's binding API, thus mimicking what the actual scheduler does.
- In the binding object, you specify a target node with the name of the node, then send a POST requets to the pod's binding API with the data set to the binding object in a JSON format.

```yaml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node02
```

- COnvert the YAML file into its equivalent JSON form.

```
curl --header "Content-Type: application/json" --request POST --data '{}' http://$SERVE
```

### 2. Label and Selector

- Labels and selectors are a standard method to group things together.
- Labels are properties attached to each item. So you add properties to each item for their class, kind and color.
- Selectors help you filter these items.
 
- In a pod definition file, under metadata, create a section called labels. Under that, add the labels ina  key value format like this. You can add as many labels as you like.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: imple-webapp
  labels:
    app: App1
    Function: Front-end
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
      - containerPort: 8080
```

- Then, can get pod with the selector option and specify the condition:

```
kubectl get pods --selector app=App1
```

- Kubernetes objects use labels and selectors internally to connect differetn objects together. 


- Annotations are used to record other details for informationpurposes. For example, tool details like name, version, ... that may be used for some kind of integration purpose. 


### 3. Taints and Tolerations
- What pods are placed on what nodes???
- There are 2 conditions whether pod can run on node:
  - First: the taint on the node
  - Second: the pod's toleration levtl to that particular taint.

- In K8s, taints and tolerations are used to set restrictions on what pods cna be scheduled on a node.

- Exmaple: we have 3 Node: node1, node2, node3 and 4 pods named pod1, pod2, pod3, pod4.
  - If we don't define any restriction, K8s scheduler will place the pods across all of the nodes to balance them out equally. 
  - Now let us assume that we have dedicated resources on node one for a particular use case or app. SO we would like only those pods that belong to this application to the placed on node1.
    - First, we prevent all pods from being placed on the node by placing a taint on the node. By default, pods have no tolerations, which means unless specified, none of the pods can tolerate any taint. So in this case, none of the pods can be placed on node1
    - Now, we add tolerantion to PodD, when the scheduler tries to place this pod on Node1, it goes through. Node1 can now only accept pods that can tolerate the taint.

--> Taints are set on nodes, tolerations are set on pods.

![taints-tolerations](/images/taint-toleration.png)

#### How to define taint and toleration

1. Taints - NODES
- We use kubeclt taint node command:
```
kubectl taint nodes node-name key=value:taint-effect


kubeclt taint nodes node1 app=myappp:NoSchedule
```

- The taint effect defines what would happen to the pods if thet do not tolerate the taint. There are three taint effects:
  - NoSchedule: the pods will not be scheduled on the node.
  - PreferNoSchedule: the system will try to avoid placing a pod on the node, but that is not guaranteed.
  - NoExecute: new pods will not be scheduled on the node and existing pods on the node, if any, will be evicted if theyy do not tolerate the taint. These pods may have been scheduled on the node before the taint was applied to the node.

2. Tolerations - PODS

- To define toleration in pods, we add in the spec of pods.

- Exmaple:
  - The taint of node: `kubectl taint nodes node1 app=blue:NoSchedule`
  - Move values in the pod definition file:
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: myapp-pod
    spec:
      containers:
      - name: nginx-container
        image: nginx
      tolerations:
      - key: "app"
        operator: "Equal"
        value: "blue"
        effect: "NoSchedule"
  ```

- Taints and tolerations only tell the node to only accept pods with certion tolerations, not tell the pod to go to a particular node.

--> When init cluster, the taint is automaticallt set for master node that prevents any pods from being scheduled on this node.

```
kubectl describe node kubemaster | grep Taint
```

- Remove taint:
```
kubectl taint nodes <node-name> <key>:<value>-

kubectl tiant nodes node 01 app:NoSchedule-
```

### 4. Node Selectors

- Case: each node have other limit resources. By default, any pods can go to any nodes. So, with heavy workload can be placed node which don't have enough resources.

--> we can set a limitation on the pods so that thay only run on particular nodes. There are two ways to do this.
  - First: using node selectors. we set in the definition file.

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: myapp
  spec:
    contianers:
    - name: data
      image: data
    nodeSelector:
      size: Large
  ```
  - To use labels in a node selector, you must have first labeled your nodes prior to creating this pod. To label for node, use command
  ```
  kubectl label nodes <node-name> <label-key>=<label-value>

  kubectl label nodes node01 size=Large
  ```

### 5. Node affinity

- The node affinity feature is to ensure that pods are hosted on particular nodes. It provides us with advanced capabilities to limit pod placement on specific nodes.

- Define:
```yaml

apiVersion: v1
kind: Pod
metadata:
name: myapp
spec:
  contianers:
  - name: data
    image: data
  affinity:
    nodeAffinity:
      requiredDuringsSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: NotIn
            values:
            - Small
```
 
 - Type Node affinity:
    - Available:
      - requiredDuringSchedulingIgnoredDuringExecution
      - preferredDuringSchedulingIgnoredDuringExecution
    - Planed:
      -  requiredDuringSchedulingRequiredDuringExecution


-  DuringScheduling is the state where a pod does not exist and is created for the first time. 


### 6. Resource Limits

- Scheduler use the resources limit to place the pod on a node, it used these numbers to identify a node which has sufficinet amount of resources available. 

- we need define in the definition file:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
  labels:
    name: app
spec:
  contianers:
  - naem: app
    image: app
    ports:
      - contianerPort: 8080
    resources:
      requests:
        memory: "1Gi"
        cpu: 2
      limits:
        memory: "2Gi"
        cpu: 2
```

- We can limit the resource the pod can use.
- The pod can not use more CPU resources than its limit. However, this is not the case with memory. A contianer can use more memory than its limit. Node can be OOM

- Ideal setup for CPU: Request and No Limits

- Use the limit ranges to define default values to be set for containers in pods that are created without a request or milit specified in the pod. This is applicable at the namespace kevel. We can define by definition file.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default:
      cpi: 500m
    defaultRequest:
      cpu: 500m
    max:
      cpu: "1"
    min:
      cpu: 100m
    type: Container
```

- It does not affect existing pods. It will only affect newer pods.


- We use ResourceQuotas to set hard limits for request and limits

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    requests.cpu: 4
    requests.memory: 4Gi
    limits.cpu: 10
    limits.memory: 10Gi
```
