### 1. Update and rollback deployment
- Rollout command to see history of rollout
```
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
```

- To see the revisions and history of our deployment.
- THe are 2 types of deploymet strategies.
  - Recreate: you destroy old version then recreate new deployment with newer image --> the application is down and inaccessable to users. 
  - Rolling update: we do not destroy all of them at once. Instead, we take down the older version and bring up a newer version one by one. This way the app never goes down and the upgrade is seamless. It is the default deployment strategy. 


- To update, we run the kubectl apply command to apply the changes. A new rollout is triggered and a new revision of the deployment is created. 
- Another way, we could use the kubectl set image command to update the image of appp. 
- To compare 2 strategies, we run the kubectl describe deployment command to see the detailed information. You will notice when the recreate strategy was used, the events indicate that the old replicaset was scaled down to zero first and then the new replicaset scaled up to five.
- Rolling update was used, the old replicaset was scaled down one at a time. 
![recreateandrollingupdate](/images/recreatevsrollingupdate.png)


- Kubernetes deployments allow you to roll back to a previous revision. To undo a change, run the commmand kubectl rollout undo deployment command, followed by the name of the deployment.
- The deployumetn will then destroy the pods in the new ReplicaSet and bring the older ones up the older ReplicaSet adn your app is back to its older format. 

![rollback](/images/rollback.png)

- Commands:

![command](/images/command.png)


### 2. ENV Variables in K8s
- ENV property starts with a dash indicating an item in the array. Each item has a pair name and value

- ENV value Types:
  - Plain Key Value: 
  - ConfigMap: 
  - Secrets:

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef: <ConfigMap>
      secretKeyRef: <Secrets>
```

### 3. ConfigMaps

- ConfigMaps are used to pass configuration datain the form of key-value pairs in Kubernetes.
- When a pod is created, inject the ConfigMap into the pod. So the key-value pairs are available as environment variables for the application hosted inside the container in the pod.
- There are two phases involved in configuring ConfigMaps.
  - First: create ConfifMap
  - Second: inject into Pod

![ConfigMaps](/images/configmaps.png)

- There two ways to create configmaps:
  - Imperative: use kubectl create configmap
  - Declarative: use kubectl create -f <"file">


- For example:
  - kubectl create configmap "config-name" --from-literal="key"="value"

- The --from-literal option is used to specify the key-values pairs in the command itself.

- You can use the --from-file option to specify a path to the file that contains the required data form this file is read and stored under the name of the file.

- Declarative way:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

kubectl create -f config-map.yml

kubectl get configmaps

kubectl describe configmaps


### 4. Secrets
- Secrets are used to store sensitive information that they're stored in an encoded or hashed format.
- Same as the configmap, there are two phases create secret adn inject into Pod.
- Create secret:
  - Kubectl create secret generic "secret-name" --from-literal="key"="value"
  - Use secret-definition.ymal

- Data must be encoded: 
  - Encode: echo -n 'mysql' | base64
  - Decode: echo -n 'vlaue' | base64 --decode

![secrets](/images/secrets.png)

### 5. Autoscaling

- Vertical scaling refers to adding more resources to an existing application.
- Horizontal scaling refers to adding more instances or more servers to your system.
- In k8s, we have sacling Cluster Infra and Scaling Workloads.
![autoscaling](/images/autosacling.png)



#### 5.1. Horizontal Pod Autoscaler (HPA)

- COntinuously monitors the metrics as we did manually using the top command.
- Then it automatically increases or decreases the number of pods in a deployment, statefulset or replicaset based on the CPU, memory or custom metrics.
- If the CPU or memory usage goes too high, HPA creates more pods to handle that, and if it drops, it removes the extra pods to save resources and thus balances the threshgholds. it can also track multiple different types of metrics which we will refer to in few minutes.

![HPA](/images/HPA.png)

![HPA-2](/images/hpa-2.png)

![hpa-3](/images/hpa-3.png)

![hpa-4](/images/hpa-4.png)


#### 5.2. Vertical Pod Autoscaler (VPA)

![vpa-1](/images/vpa-1.png)

![compare-hpa-vpa](/images/compare-hpa-vpa.png)