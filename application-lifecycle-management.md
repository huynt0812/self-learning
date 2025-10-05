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


### 2. Commands and arguments in Docker
