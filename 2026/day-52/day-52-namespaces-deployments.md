Ouestion 1  -What namespaces are and why you would use them?
Answer - Namespaces are logical partitions that divide a single physical Kubernetes cluster into multiple independent virtual clusters.

Why use them?

Environment Isolation: Safely run dev, staging, and prod environments on the exact same underlying hardware.

Prevent Name Collisions: You can have a pod named database in the dev namespace and another pod named database in the prod namespace without them conflicting.

Resource Limits (Quotas): You can cap the total CPU and RAM available to a specific namespace, ensuring one team's buggy code doesn't drain the entire cluster's resources.

Security (RBAC): You can restrict user permissions geographically. For example, giving a developer full admin rights inside the dev namespace, but zero access to the prod namespace.

Question 2 - Your Deployment manifest and an explanation of each section?
Answer -apiVersion & kind: Defines what you are building (Deployment) and which API group it belongs to (apps/v1).
metadata: The name of the Deployment resource itself.
spec.replicas: The exact number of identical Pods you want running.
spec.selector: How the Deployment identifies which Pods it manages. It actively searches the cluster for Pods tagged with app: web.
template.metadata: The actual labels applied to every new Pod created. (This must match the selector above).
template.spec: The Pod blueprint. It defines the list of containers (note the - dash), the container image, and the ports to open.

Question 3 - What happens when you delete a Pod managed by a Deployment vs a standalone Pod
Answer - pod gets created immediatetly after deletion with new IP assigned The Deployment's underlying ReplicaSet constantly monitors the cluster state. If you delete one of its pods, it notices the actual count is lower than your desired count and immediately spins up a brand-new replacement to replace it. and in standalone pod gets permanently deleted as it is ephemeral

Question 4 -How scaling works (both imperative and declarative)
Answer - Imperative Scaling (The Command Line)
You change the number of running pods instantly on the live cluster. This is fast for debugging but isn't saved in your code repository.

The Command: kubectl scale deployment my-app --replicas=5

Declarative Scaling (The YAML File)
You update your configuration file with the new desired number of pods, then apply the file. This is best practice for production because your code history matches the live infrastructure.

The Action: Edit your deployment.yaml and change replicas: 3 to replicas: 5.

The Command: kubectl apply -f deployment.yaml

Regardless of the method used, the Kubernetes Controller instantly detects the difference between the actual state and the desired state, and spins up (or terminates) the exact number of pods needed to match.

Question 5 - How rolling updates and rollbacks work
Answer - Rolling Updates (Zero Downtime Deployments)
When you update your Deployment (like changing the container image from v1 to v2), Kubernetes replaces your pods gradually to prevent downtime.The Process: It creates a new, secondary ReplicaSet. It spins up a new v2 pod, waits for it to become READY and accept traffic, and only then terminates an old v1 pod.The Result: It repeats this handoff one-by-one (or in small batches) until all old pods are replaced, ensuring the application stays online the entire time.Rollbacks (Reverting Bad Updates)
If you deploy a broken image (like one that triggers a CrashLoopBackOff), Kubernetes retains the configuration of your previous ReplicaSets so you can instantly abort and revert.The Command: kubectl rollout undo deployment my-appThe Process: Kubernetes immediately stops the broken rollout, scales the new, failing ReplicaSet down to zero, and scales your previous, known-good ReplicaSet back up to full capacity.
Screenshot of your Deployment and Pods running
