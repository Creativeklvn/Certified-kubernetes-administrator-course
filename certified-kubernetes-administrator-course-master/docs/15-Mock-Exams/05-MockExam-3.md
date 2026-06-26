# Mock Exam 3
  
  Take me to [Mock Exam 3](https://kodekloud.com/topic/mock-exam-3-2/)

#### Solution to the Mock Exam 3

1. You are an administrator preparing your environment to deploy a Kubernetes cluster using kubeadm. Adjust the following network parameters on the system to the following values, and make sure your changes persist reboots:

net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1


  - net.ipv4.ip_forward is set to 1

  - net.bridge.bridge-nf-call-iptables is set to 1

      <details>

      </details>

2. Create a new service account with the name `pvviewer`. Grant this Service account access to `list` all PersistentVolumes in the cluster by creating an appropriate cluster role called `pvviewer-role` and ClusterRoleBinding called `pvviewer-role-binding`.
Next, create a pod called `pvviewer` with the image: `redis` and serviceAccount: `pvviewer` in the default namespace.

- ServiceAccount: pvviewer

- ClusterRole: pvviewer-role

- ClusterRoleBinding: pvviewer-role-binding

- Pod: pvviewer

- Is the pod configured to use ServiceAccount pvviewer?
  
      <details>

      
      </details>

 
3. Create a StorageClass named `rancher-sc` with the following specifications:

The provisioner should be `rancher.io/local-path`.
The volume binding mode should be `WaitForFirstConsumer`.
Volume expansion should be enabled.


  - StorageClass rancher-sc is present

  - Provisioner is rancher.io/local-path

  - VolumeBindingMode is WaitForFirstConsumer

      <details>
          
      
      </details>
  
4. Create a ConfigMap named `app-config` in the namespace 
`cm-namespace` with the following key-value pairs:

```shell
ENV=production
LOG_LEVEL=info
```

Then, modify the existing Deployment named `cm-webapp` in the same namespace to use the `app-config` ConfigMap by setting the environment variables `ENV` and `LOG_LEVEL` in the container from the ConfigMap.


- ConfigMap app-config is created

- Deployment uses the app-config ConfigMap for variable ENV and LOG LEVEL

- Are the environment variables reflected in the deployment?

- ConfigMap has proper ENV value

- ConfigMap has proper LOG_LEVEL value
  
      <details>
      

      </details>

5. Create a PriorityClass named `low-priority` with a value of 50000. A pod named `lp-pod` exists in the namespace `low-priority`. Modify the pod to use the priority class you created. Recreate the pod if necessary.


- Is the PriorityClass low-priority created?

- Low priority class value is set properly to 50000

- Pod lp-pod uses the low-priority PriorityClass

    <details>

  </details>

6. A pod called `np-test-1` and a service called `np-test-service` have been deployed in the `default` namespace. A `default-deny` NetworkPolicy is currently blocking all ingress traffic to pods in this namespace, which is why the service is unreachable.

Create a new NetworkPolicy named `ingress-to-nptest` in the `default` namespace that allows ingress traffic from all sources to the `np-test-1` pod on port `80`.


Important: Don't delete any current objects deployed.

- Important: Don't Alter Existing Objects! (default-deny NetworkPolicy must still exist)

- NetworkPolicy: Is the port correct?

- NetworkPolicy: Is it applied to the correct Pod?

- Is ingress traffic to np-test-1 pod reachable on port 80?
      
    <details>

    

    </details>

7. Taint the worker node `node01` to be Unschedulable. Once done, create a pod called `dev-redis`, image `redis:alpine`, to ensure workloads are not scheduled to this worker node. Finally, create a new pod called `prod-redis` and image: `redis:alpine` with toleration to be scheduled on `node01`.


key: `env_type`, value: `production`, operator: `Equal` and effect: `NoSchedule`

- key = env_type

- value = production

- effect = NoSchedule

- Is pod 'dev-redis' (no tolerations) not scheduled on node01?

- Is the 'prod-redis' to running on node01?

      <details>

      </details>

8. A PersistentVolumeClaim named `app-pvc` exists in the namespace `storage-ns`, but it is not getting bound to the available PersistentVolume named `app-pv`.

Inspect both the PVC and PV and identify why the PVC is not being bound and fix the issue so that the PVC successfully binds to the PV. Do not modify the PV resource.

- Is PVC correctly bound to PV?

    <details>

    </details>

9. A kubeconfig file called `super.kubeconfig` has been created under `/root/CKA`. There is something wrong with the configuration. Troubleshoot and fix it.


- Fix /root/CKA/super.kubeconfig

      <details>

      </details>

10. We have created a new deployment called `nginx-deploy`. Scale the deployment to 3 replicas. Has the number of replicas increased? Troubleshoot and fix the issue.


- Does the deployment have 3 replicas?

      <details>
      
      </details>

11. Create a Horizontal Pod Autoscaler (HPA) `api-hpa` for the deployment named `api-deployment` located in the `api` namespace.
The HPA should scale the deployment based on a custom metric named `requests_per_second`, targeting an average value of 1000 requests per second across all pods.
Set the minimum number of replicas to 1 and the maximum to 20.

Note: Deployment named `api-deployment` is available in api namespace. Ignore errors due to the metric `requests_per_second` not being tracked in `metrics-server`


- Is `api-hpa HPA` deployed in api namespace?

- Is api-hpa configured for metric `requests_per_second`?

    <details>
    

      </details>

12. Configure the `web-route` to split traffic between `web-service` and `web-service-v2`.The configuration should ensure that 80% of the traffic is routed to `web-service` and 20% is routed to `web-service-v2`.

Note: `web-gateway`, `web-service`, and `web-service-v2` have already been created and are available on the cluster.


- Is the web-route deployed as HTTPRoute?

- Is the route configured to gateway `web-gateway`?

- Is the route configured to service `web-service`?

    <details>
    

      </details>

13. One application, `webpage-server-01`, is currently deployed on the Kubernetes cluster using Helm. A new version of the application is available in a Helm chart located at `/root/new-version`.

Validate this new Helm chart, then install it as a new release named `webpage-server-02`. After confirming the new release is installed, uninstall the old release `webpage-server-01`.

- Is the new version app deployed?

- Is the old version app uninstalled?

    <details>
    

      </details>

14. While preparing to install a CNI plugin on your Kubernetes cluster, you typically need to confirm the cluster-wide Pod network CIDR. Identify the Pod subnet configured for the cluster (the value specified under `podSubnet` in the kubeadm configuration). Output this CIDR in the format `x.x.x.x/x` to a file located at `/root/pod-cidr.txt`.


Note: Use the cluster-wide `podSubnet` from the `kubeadm-config` ConfigMap, not the per-node CIDR from `kubectl get node`.

- Is the cluster-wide Pod CIDR network correctly written to the file?

    <details>
    

      </details>
