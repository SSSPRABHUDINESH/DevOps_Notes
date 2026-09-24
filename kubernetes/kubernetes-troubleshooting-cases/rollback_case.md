
## Refined Question for Your Notes
# Title: How does Kubernetes deployment rollback work vs. Git version control?
Question:
I deployed Version 1 of an application using a Kubernetes deployment manifest. Later, I updated the manifest and deployed Version 2. Because Version 2 is experiencing runtime failures, I need to immediately roll back the cluster to Version 1.

   1. What are the exact kubectl commands required to safely execute this rollback?
   2. Does a Kubernetes rollback mean I have to manually pull the Version 1 source code from GitHub and redeploy it, or does Kubernetes handle the version reversion natively within the cluster?
   3. What happens to the underlying cluster resources (like ReplicaSets) behind the scenes during this rollback process?

Ans: 
To roll back to version 1 in Kubernetes, you do not need to pull the code from GitHub and redeploy it manually.
Kubernetes automatically keeps a history of your previous deployments right inside the cluster. When you roll back, Kubernetes simply changes the running container images and configurations back to the exact state they were in during version 1.
Here is exactly how to do it using the command line.
## 1. Check your deployment history
First, look at the available revisions to find your version 1. Run this command (replace your-deployment-name with the actual name of your deployment):

```yaml
kubectl rollout history deployment/your-deployment-name
```

You will see a list of revision numbers.

* Revision 1 usually corresponds to your first deployment (Version 1).
* Revision 2 corresponds to your second deployment (Version 2).

## 2. Roll back to Version 1
To instantly revert to your first version, run the rollout command with the specific revision number:
```yaml
kubectl rollout undo deployment/your-deployment-name --to-revision=1
```

(Note: If you just want to roll back to the immediate previous version, you can simply run kubectl rollout undo deployment/your-deployment-name without the revision flag).
## 3. Track the status
You can watch the rollback happen in real-time to ensure the old pods come back up successfully:
```yaml
kubectl rollout status deployment/your-deployment-name
```

------------------------------
## What happens behind the scenes?

* ReplicaSets: When you deployed Version 2, Kubernetes did not delete the underlying configuration (ReplicaSet) for Version 1; it just scaled it down to 0 copies.
* The Rollback: When you run the undo command, Kubernetes scales the Version 2 ReplicaSet down to 0 and scales the Version 1 ReplicaSet back up.

Important Best Practice: While the cluster rollback fixes the immediate production failure, remember to update your local manifest files or your GitHub repository afterward so your code history matches what is actually running in production!
If you run into any issues during the rollback, let me know:

