# Mock Exam 2

  Test My Knowledge, Take me to [Mock Exam 2](https://kodekloud.com/topic/mock-exam-2-3/)

  #### Solution to the Mock Exam 2

1. Create a StorageClass named `local-sc` with the following specifications and set it as the default storage class:

    - The provisioner should be `kubernetes.io/no-provisioner`
    - The volume binding mode should be `WaitForFirstConsumer`
    - Volume expansion should be enabled

      <details>

      Check kubernetes.io documentation on StorageClasses

      Create the StorageClass YAML file:
      ```yaml
      apiVersion: storage.k8s.io/v1
      kind: StorageClass
      metadata:
        name: local-sc
        annotations:
          storageclass.kubernetes.io/is-default-class: "true"
      provisioner: kubernetes.io/no-provisioner
      volumeBindingMode: WaitForFirstConsumer
      allowVolumeExpansion: true
      ```

      Apply the manifest on the specified cluster:

      ```bash
      kubectl apply -f local-sc.yaml
      ```

      Verify the StorageClass

      ```bash
      kubectl get storageclass
      ```

      You should see `local-sc` marked as (default) with the correct provisioner and settings
      </details>

2. Create a deployment named `logging-deployment` in the namespace `logging-ns` with 1 replica, with the following specifications:

    - The main container should be named `app-container`, use the image `busybox`, and should run the following command to simulate writing logs:

    ```bash
    sh -c "while true; do echo 'Log entry' >> /var/log/app/app.log; sleep 5; done"
    ```

    - Add a sidecar container named `log-agent` that also uses the `busybox` image and runs the command:

    ```bash
    tail -f /var/log/app/app.log
    ```

    - log-agent logs should display the entries logged by the main `app-container`
  
      <details>

      Ensure both containers mount the same emptyDir volume.

      Create the Deployment YAML - example YAML file below:

      ```yaml
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: logging-deployment
        namespace: logging-ns
      spec:
        replicas: 1
        selector:
          matchLabels:
            app: logger
        template:
          metadata:
            labels:
              app: logger
          spec:
            volumes:
              - name: log-volume
                emptyDir: {}
            initContainers:
              - name: log-agent
                image: busybox
                command: ['sh', '-c', 'touch /var/log/app/app.log; tail -f /var/log/app/app.log']
                volumeMounts:
                  - name: log-volume
                    mountPath: /var/log/app
                restartPolicy: Always 
            containers:
              - name: app-container
                image: busybox
                command: ['sh', '-c', 'while true; do echo "Log entry" >> /var/log/app/app.log; sleep 5; done']
                volumeMounts:
                  - name: log-volume
                    mountPath: /var/log/app
      ```

      Apply the deployment:

      ```bash
      kubectl apply -f logger-deployment.yaml
      ```

      Verify that the log-agent container displays log entries:

      ```bash
      kubectl logs -n logging-ns deployment/logging-deployment -c log-agent
      ```

      You should see repeated `Log entry` lines in the output

      </details>

 
3. A Deployment named `webapp-deploy` is running in the `ingress-ns `namespace and is exposed via a Service named webapp-svc.

    Create an Ingress resource called `webapp-ingress` in the same namespace that will route traffic to the service. The Ingress must:

    - Use `pathType: Prefix`
    - Route requests sent to path `/` to the backend service
    - Forward traffic to port `80` of the service
    - Be configured for the host `kodekloud-ingress.app`

      Test app availablility using the following command:

      ```bash
      curl -s http://kodekloud-ingress.app/
      ```

      <details>
          
      Make sure the path is set correctly and the pathType is Prefix. Use host-based routing

      Checkout the resources in the `ingress-ns` namespace:

      ```bash
      kubectl get deployment webapp-deploy -n ingress-ns
      kubectl get svc webapp-svc -n ingress-ns
      ```

      Create the Ingress YAML file:

      ```yaml
      apiVersion: networking.k8s.io/v1
      kind: Ingress
      metadata:
        name: webapp-ingress
        namespace: ingress-ns
      spec:
        ingressClassName: nginx
        rules:
        - host: kodekloud-ingress.app
          http:
            paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: webapp-svc
                  port:
                    number: 80
      ```

      Apply the Ingress resource:

      ```yaml
      kubectl apply -f webapp-ingress.yaml
      ```

      Test access to the app:

      ```bash
      curl http://kodekloud-ingress.app/
      ```

      </details>
  
4. Create a new deployment called `nginx-deploy`, with image `nginx:1.16` and `1` replica. Next, upgrade the deployment to version `1.17` using rolling update.

    Note: Use the `kubectl apply` command to create or update the deployment.
  
      <details>
      Explore the `--record` option while creating the deployment while working with the deployment definition file. Then make use of the `kubectl apply` command to create or update the deployment.

      To create a deployment definition file `nginx-deploy`:

      ```bash
      kubectl create deployment `nginx-deploy` --image=nginx:1.16 --dry-run=client -o yaml > deploy.yaml
      ```

      To create a resource from definition file and to record:

      ```bash
      kubectl apply -f deploy.yaml --record
      ```

      To view the history of deployment `nginx-deploy`:

      ```bash
      kubectl rollout history deployment `nginx-deploy`
      ```

      To upgrade the image to next given version:

      ```bash
      kubectl set image deployment/`nginx-deploy` nginx=nginx:1.17 --record
      ```

      To view the history of deployment `nginx-deploy`:

      ```bash
      kubectl rollout history deployment nginx-deploy
      ```

      </details>

5. Create a new user called `john`. Grant him access to the cluster using a csr named `john-developer`. Create a role `developer` which should grant John the permission to `create`, `list`, `get`, `update` and `delete ``pods` in the `development` namespace. The private key exists in the location: `/root/CKA/john.key and csr at /root/CKA/john.csr`.

    Important Note: As of kubernetes 1.19, the CertificateSigningRequest object expects a `signerName`.

    Please refer to the documentation to see an example. The documentation tab is available at the top right of the terminal.

    <details>

    To create a CSR as follows:

    ```yaml
    apiVersion: certificates.k8s.io/v1
    kind: CertificateSigningRequest
    metadata:
      name: john-developer
    spec:
      signerName: kubernetes.io/kube-apiserver-client
      request: LS0t...
      usages:
      - digital signature
      - key encipherment
      - client auth
    ```

    To approve this certificate, run:

    ```bash
    kubectl certificate approve john-developer
    ```

    Next, create a role developer and rolebinding developer-role-binding, run the command:

    ```bash
    kubectl create role developer --resource=pods --verb=create,list,get,update,delete --namespace=development
    ```

    or

    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: developer
      namespace: development
    rules:
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["create", "list", "get", "update", "delete"]
    ```

    ```bash
    kubectl apply -f rol.yaml
    ```

    Next, create a role developer and rolebinding developer-role-binding, run the command:
    
    ```bash
    kubectl create rolebinding developer-role-binding --role=developer --user=john --namespace=development
    ```

    or

    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: john-developer-binding
      namespace: development
    subjects:
    - kind: User
      name: john
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: Role
      name: developer
      apiGroup: rbac.authorization.k8s.io
    ```

    ```bash
    kubectl apply -f rolebinding.yaml
    ```

    To verify the permission from kubectl utility tool:

    ```bash
    kubectl auth can-i update pods --as=john --namespace=development
    ```

    </details>

6. Create an nginx pod named `nginx-resolver` using the `nginx` image and expose it internally using a `ClusterIP` service called `nginx-resolver-service`.

    From within the cluster, verify:

    - DNS resolution of the service name
    - Network reachability of the pod using its IP address

    Use the `busybox:1.28` image to perform the lookups.

    Save the service DNS lookup output to `/root/CKA/nginx.svc` and the pod IP lookup output to `/root/CKA/nginx.pod`.
      
    <details>

    Use the command `kubectl run` and create a nginx pod and busybox pod. Resolve it, nginx service and its pod name from `busybox` pod.

    To create a pod `nginx-resolver` and expose it internally:

    ```bash
    kubectl run nginx-resolver --image=nginx
    kubectl expose pod nginx-resolver --name=nginx-resolver-service --port=80 --target-port=80 --type=ClusterIP
    ```

    To create a pod `test-nslookup`. Test that you are able to look up the service and pod names from within the cluster:

    ```bash
    kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service
    kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service > /root/CKA/nginx.svc
    ```

    Get the IP of the nginx-resolver pod and replace the dots(.) with hyphon(-) which will be used below.

    ```bash
    kubectl get pod nginx-resolver -o wide
    kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup <P-O-D-I-P.default.pod> > /root/CKA/nginx.pod
    ```

    </details>

7. Create a static pod on node01 called nginx-critical with the image nginx. Make sure that it is recreated/restarted automatically in case of a failure.

    For example, use /etc/kubernetes/manifests as the static Pod path.

      <details>

      To create a static pod called nginx-critical by using below command:

      ```bash
      kubectl run nginx-critical --image=nginx --dry-run=client -o yaml > static.yaml
      ```

      Copy the contents of this file or use `scp` command to transfer this file from `controlplane` to `node01` node.

      ```bash
      root@controlplane:~# scp static.yaml node01:/root/
      ```

      To know the IP Address of the `node01` node:

      ```bash
      root@controlplane:~# kubectl get nodes -o wide
      # Perform SSH
      root@controlplane:~# ssh node01
      OR
      root@controlplane:~# ssh <IP of node01>
      ```

      On `node01` node:

      Check if static pod directory is present which is `/etc/kubernetes/manifests`, if it's not present then create it.

      ```bash
      root@node01:~# mkdir -p /etc/kubernetes/manifests
      ```

      Add that complete path to the `staticPodPath` field in the kubelet `config.yaml` file.

      ```bash
      root@node01:~# vi /var/lib/kubelet/config.yaml
      ```

      now, move/copy the static.yaml to path /etc/kubernetes/manifests/.

      ```bash
      root@node01:~# cp /root/static.yaml /etc/kubernetes/manifests/
      ```

      Go back to the `controlplane` node and check the status of static pod:

      ```BASH
      root@node01:~# exit
      logout
      root@controlplane:~# kubectl get pods
      ```

      </details>

8. Create a Horizontal Pod Autoscaler with name `backend-hpa` for the deployment named `backend-deployment` in the backend namespace with the `webapp-hpa.yaml` file located under the root folder.
Ensure that the HPA scales the deployment based on memory utilization, maintaining an average memory usage of 65% across all pods.
Configure the HPA with a minimum of 3 replicas and a maximum of 15.

    <details>

    Volume name: `pv-analytics`
    Storage: `100Mi`
    Access mode: `ReadWriteMany`
    Host path: `/pv/data-analytics`

    Apply the below manifest to create a PV:

    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: pv-analytics
    spec:
      capacity:
        storage: 100Mi
      volumeMode: Filesystem
      accessModes:
        - ReadWriteMany
      hostPath:
        path: /pv/data-analytics
    ```

    </details>

9. Modify the existing `web-gateway` on `cka5673` namespace to handle HTTPS traffic on port `443` for `kodekloud.com`, using a TLS certificate stored in a secret named `kodekloud-tls`.

    - Is the web gateway configured to listen on the hostname kodekloud.com?

    - Is the HTTPS listener configured with the correct TLS certificate?

      <details>
      Check the configuration of the web-gateway.

      ```bash
      kubectl get gateway web-gateway -n cka5673 -o yaml
      ```

      ```yaml
      apiVersion: gateway.networking.k8s.io/v1
      kind: Gateway
      metadata:
        name: web-gateway
        namespace: cka5673
        resourceVersion: "21919"
        uid: a1d0e35d-5126-4000-88ec-f440941eed75
      spec:
        gatewayClassName: kodekloud
        listeners:
        - allowedRoutes:
            namespaces:
              from: Same
          name: https
          port: 80
          protocol: HTTP
      ```

      The current configuration of the `web-gateway` is incorrect as it is listening on port 80 using the HTTP protocol. To update the `web-gateway` to listen on port 443 with the TLS certificate, use the following manifest:

      ```yaml # web-gateway.yaml
      apiVersion: gateway.networking.k8s.io/v1
      kind: Gateway
      metadata:
        name: web-gateway
        namespace: cka5673
      spec:
        gatewayClassName: kodekloud
        listeners:
          - name: https
            protocol: HTTPS
            port: 443
            hostname: kodekloud.com
            tls:
              certificateRefs:
                - name: kodekloud-tls
      ```

      Apply the configuration as follows:

      ```bash
      kubectl apply -f web-gateway.yaml     
      ```

      </details>

10. On the cluster, the team has installed multiple helm charts on  a different namespace. By mistake, those deployed resources include one of the vulnerable images called kodekloud/webapp-color:v1. Find out the release name and uninstall it.


    - Is helm release uninstalled?

      <details>
      In this task, we will use the helm commands and `jq` tool. Here are the steps: -

      Run the `helm ls` command with `-A` option to list the releases deployed on all the namespaces using helm.

      ```bash
      helm ls -A
      ```

      We will use the `jq` tool to extract the image name from the deployments.

      ```bash
      kubectl get deploy -n <NAMESPACE> <DEPLOYMENT-NAME> -o json | jq -r '.spec.template.spec.containers[].image'
      ```

      Replace `<NAMESPACE>` with the namespace and `<DEPLOYMENT-NAME>` with the deployment name, which we get from the previous commands.

      After finding the `kodekloud/webapp-color:v1` image, use the `helm uninstall` to remove the deployed chart that are using this vulnerable image.

      ```bash
      helm uninstall <RELEASE-NAME> -n <NAME
      ```

      </details>

11. You are requested to create a NetworkPolicy to allow traffic from frontend apps located in the frontend namespace, to backend apps located in the backend namespace, but not from the databases in the databases namespace. There are three policies available in the /root folder. Apply the most restrictive policy from the provided YAML files to achieve the desired result. Do not delete any existing policies.


    - Correct NetworkPolicy applied

    - Incorrect NetworkPolicy is not applied

    - Second incorrect NetworkPolicy is not applied

    <details>
    Read through all provided NetworkPolicy YAML files carefully. Only one of them restricts traffic from the `databases` namespace while allowing it from `frontend`.

    On controlplane, Review the contents of the three YAML files:

    ```bash
    cat /root/net-pol-1.yaml
    cat /root/net-pol-2.yaml
    cat /root/net-pol-3.yaml
    ```

    Understand the differences:
    - `net-pol-1.yaml`: Too broad; allows traffic from any namespace with a certain label.
    - `net-pol-2.yaml`: Incorrect; explicitly allows both `frontend` and `databases`.
    - `net-pol-3.yaml`: Correct; only allows traffic from the `frontend` namespace.

    Apply the correct policy (`net-pol-3.yaml`):

    ```bash
    kubectl apply -f /root/net-pol-3.yaml
    ```

    Verify it’s the only one applied:

    ```bash
    kubectl get netpol -n backend
    ```

    You should only see `net-policy-3` listed
      </details>
