# Scenario: "Bilbao": Basic Kubernetes Problems

**Level:** Easy  
**Tags:** `kubernetes`, `k8s`, `scheduling`, `orchestration`, `troubleshooting`  

## 📝 Problem Description
A Kubernetes Deployment configured with an Nginx pod and a LoadBalancer service is failing to serve traffic. The pod is permanently stuck in a `Pending` state, preventing the LoadBalancer from routing external traffic to the application.

## 💡 The Solution Strategy
A `Pending` status indicates that the Kubernetes scheduler cannot find a suitable node that satisfies all constraints declared in the pod configuration template. Troubleshooting requires inspecting the pod's scheduler lifecycle events, checking worker node capacities/labels, and aligning the manifest requirements with the physical cluster architecture.

### The Commands
```bash
# 1. Audit cluster object states to see the scheduling bottleneck
kubectl get pods
# Output reveals the nginx pod is stuck in a 'Pending' state

# 2. Inspect the internal scheduler event stream for the blocked pod
kubectl describe pod nginx-deployment-67699598cc-zrj6f
# Error reveals: "0/2 nodes are available: 1 node(s) didn't match Pod's node affinity/selector..."

# 3. Inspect active nodes and their corresponding metadata labels
kubectl get nodes --show-labels
# Output shows the only healthy control-plane/worker node ('node1') lacks the requested 'disk=ssd' label

# 4. Solution Option A: Dynamically update node labels to satisfy the scheduler match
kubectl label node node1 disk=ssd

# 5. Solution Option B: Fix the manifest file (Source of Truth)
nano /home/admin/manifest.yml
# - Removed the restrictive 'nodeSelector' block completely
# - Lowered memory limits/requests from a heavy '2000Mi' down to a lean '100Mi' to prevent resource starvation on a small VM

# 6. Apply the clean configuration blueprint
kubectl apply -f /home/admin/manifest.yml

# 7. Verify the pod transitions cleanly into service availability
kubectl get pods
# Output shows the pod is now 'Running' and '1/1 Ready'

# 8. Test end-to-end network delivery through the LoadBalancer External IP
curl 10.43.216.196
# Output returns the default "Welcome to nginx!" raw HTML page
