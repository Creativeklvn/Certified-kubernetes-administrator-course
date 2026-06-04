# Lightning Lab 1

  - I am ready! [Take me to Lightning Lab 1](https://kodekloud.com/topic/lightning-lab-1-2/)

## Solution to LL-1

1.  <details>
    <summary>Upgrade the current version of kubernetes from 1.28.0 to 1.29.0 exactly using the kubeadm utility.</summary>
    
    To seamlessly transition from Kubernetes v1.34 to v1.35 and gain access to the packages specific to the desired Kubernetes minor version, follow these essential steps during the upgrade process. This ensures that your environment is appropriately configured and aligned with the features and improvements introduced in Kubernetes v1.35.

    1. On the controlplane node:

        Use any text editor you prefer to open the file that defines the Kubernetes apt repository.

        ```bash
        vi /etc/apt/sources.list.d/kubernetes.list
        ```

    1. Update the version in the URL to the next available minor release, i.e v1.35. It should be smilar to this:

        ```bash
        deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /
        ```

    1. After making changes, save the file and exit from your text editor. Proceed with the next instruction.

        ```bash
        kubectl drain controlplane --ignore-daemonsets
        apt update
        apt-cache madison kubeadm
        ```

    1. Based on the version information displayed by apt-cache madison, it indicates that for Kubernetes version 1.35.0, the available package version is 1.35.0-1.1. Therefore, to install kubeadm for Kubernetes v1.35.0, use the following command:

        ```bash
        apt-get install kubeadm=1.35.0-1.1
        ```

    1. Run the following command to upgrade the Kubernetes cluster.

        ```bash
        kubeadm upgrade plan v1.35.0
        kubeadm upgrade apply v1.35.0
        ```

    1. Now, upgrade the version and restart Kubelet. Also, mark the node (in this case, the "controlplane" node) as schedulable.

        ```bash
        apt-get install kubelet=1.35.0-1.1
        systemctl daemon-reload
        systemctl restart kubelet
        ```

    1. Reinstate controlplane node

        ```bash
        kubectl uncordon controlplane
        ```

    1. Before draining node01, if the controlplane gets taint during an upgrade, we have to remove it.Identify the taint first.

        ```bash
        kubectl describe node controlplane | grep -i taint
        ```

    1. Remove the taint with help of "kubectl taint" command.

        ```bash
        kubectl taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule-
        ```

    1. Verify it, the taint has been removed successfully.  

        ```bash
        kubectl describe node controlplane | grep -i taint
        ```

    1. Now, drain the node01 as follows: -

        ```bash
        kubectl drain node01 --ignore-daemonsets
        ```

    1. SSH to the node01 and perform the below steps as follows:

    Use any text editor you prefer to open the file that defines the Kubernetes apt repository.

        ```bash
        vim /etc/apt/sources.list.d/kubernetes.list
        ```

    1. Update the version in the URL to the next available minor release, i.e v1.35.

        ```bash
        deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /
        ```

    1. After making changes, save the file and exit from your text editor. Proceed with the next instruction.

        ```bash
        apt update
        apt-cache madison kubeadm
        ```

    1. Based on the version information displayed by apt-cache madison, it indicates that for Kubernetes version 1.35.0, the available package version is 1.35.0-1.1. Therefore, to install kubeadm for Kubernetes v1.35.0, use the following command:

        ```bash
        apt-get install kubeadm=1.35.0-1.1
        ```

    1. Upgrade the node 

        ```bash
        kubeadm upgrade node
        ```

    1. Now, upgrade the version and restart Kubelet.

        ```bash
        apt-get install kubelet=1.35.0-1.1
        systemctl daemon-reload
        systemctl restart kubelet
        ```

    1. To exit from the specific node, type exit or logout on the terminal.

        ```bash
        exit
        ```

    1. Back on the controlplane node

        ```bash
        kubectl uncordon node01
        kubectl get pods -o wide | grep gold
        ``` # make sure this is scheduled on a node

    **TIP**

    To do cluster upgrades faster and save at least 3 minutes, you can work on both nodes at the same time.

    While `kubeadm upgrade apply` is running on `controlplane`, which takes some minutes, open a second terminal and perform steps `ii`, `iii` and `iv` of "Upgrade `node01`", so that it is ready for `kubeadm upgrade node` as soon as you have drained it.

    </details>

2.  <details>
    <summary>Print the names of all deployments in the admin2406 namespace in the following format...</summary>

    This is a job for `custom-columns` output of kubectl

    ```bash
    kubectl -n admin2406 get deployment -o custom-columns=DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[].image,READY_REPLICAS:.status.readyReplicas,NAMESPACE:.metadata.namespace --sort-by=.metadata.name > /opt/admin2406_data
    ```

    </details>

3.  <details>
    <summary> A kubeconfig file called admin. kubeconfig has been created in /root/CKA. There is something wrong with the configuration. Troubleshoot and fix it.</summary>

    1. First, let's test this kubeconfig

        ```bash
        kubectl get pods --kubeconfig /root/CKA/admin.kubeconfig
        ```

    Notice the error message.

    1. Now look at the default kubeconfig for the correct setting.

        ```bash
        cat ~/.kube/config
        ```

    1. Make the correction

        ```bash
        vi /root/CKA/admin.kubeconfig
        ```

    Make sure the port for the kube-apiserver is correct. So for this change port from 4380 to 6443.

    1. Test

        ```bash
        kubectl cluster-info --kubeconfig /root/CKA/admin.kubeconfig
        ```

  </details>

4.  <details>

    <summary>
    Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica.</summary>

    1. Create a new deployment called nginx-deploy

        ```bash
        kubectl create deployment nginx-deploy --image=nginx:1.16
        ```

    1. Next upgrade the deployment to version 1.17 using rolling update.

        ```bash
        kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record
        ```

    You may ignore the deprecation warning.


    1. Manually annotate the deployment to log the change, replacing the functionality of the deprecated --record option.

        ```bash
        kubectl annotate deployment nginx-deploy kubernetes.io/change-cause="Updated nginx image to 1.17"
        ```

    This annotation acts as a record of the image update.
    </details>

5.  <details>
    <summary>A new deployment called alpha-mysql has been deployed in the alpha namespace. However, the pods are not running. Troubleshoot and fix the issue.</summary>

    The deployment should make use of the persistent volume alpha-pv to be mounted at /var/lib/mysql and should use the environment variable MYSQL_ALLOW_EMPTY_PASSWORD=1 to make use of an empty root password.

    Important: Do not alter the persistent volume.

    1. Inspect the deployment to check the environment variable is set. Here I'm using `yq` which is like `jq` but for YAML to not have to view the _entire_ deployment YAML, just the section beneath `containers` in the deployment spec.

        ```bash
        kubectl get deployment -n alpha alpha-mysql  -o yaml | yq e .spec.template.spec.containers -
        ```

    1. Find out why the deployment does not have minimum availability. We'll have to find out the name of the deployment's pod first, then describe the pod to see the error.

        ```bash
        kubectl get pods -n alpha
        ```

    1. Step 3

        ```bash
        kubectl describe pod -n alpha alpha-mysql-xxxxxxxx-xxxxx
        ```

    1. We find that the requested PVC isn't present, so create it. First, examine the Persistent Volume to find the values for access modes, capacity (storage), and storage class name

        ```bash
        kubectl get pv alpha-pv
        ```

    1. Now use `vi` to create a PVC manifest

        ```yaml
        apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: mysql-alpha-pvc
          namespace: alpha
        spec:
          accessModes:
          - ReadWriteOnce
          resources:
            requests:
              storage: 1Gi
        storageClassName: slow
        ```

  </details>

6.  <details>
    <summary>Take the backup of ETCD at the location /opt/etcd-backup.db on the controlplane node.</summary>

    This question is a bit poorly worded. It requires us to make a backup of etcd and store the backup at the given location.

    Know that the certificates we need for authentication of `etcdctl` are located in `/etc/kubernetes/pki/etcd`

    ```bash
    ETCDCTL_API='3' etcdctl snapshot save \
      --cacert=/etc/kubernetes/pki/etcd/ca.crt \
      --cert=/etc/kubernetes/pki/etcd/server.crt \
      --key=/etc/kubernetes/pki/etcd/server.key \
      /opt/etcd-backup.db
    ```

    Whilst we _could_ also use the argument `--endpoints=127.0.0.1:2379`, it is not necessary here as we are on the controlplane node, same as `etcd` itself. The default endpoint is the local host.
    </details>

7.  <details>
    <summary>Create a pod called secret-1401 in the admin1401 namespace using the busybox image....</summary>

    The container within the pod should be called `secret-admin` and should sleep for 4800 seconds.

    The container should mount a read-only secret volume called secret-volume at the path `/etc/secret-volume`. The secret being mounted has already been created for you and is called `dotfile-secret`.

    1. Use imperative command to get a starter manifest

        ```bash
        kubectl run secret-1401 -n admin1401 --image=busybox --dry-run=client -oyaml --command -- sleep 4800 > admin.yaml
        ```

    1. Edit this manifest to add in the details for mounting the secret

        ```
        vi admin.yaml
        ```

        Add in the volume and volume mount sections seen below

        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          creationTimestamp: null
          labels:
            run: secret-1401
          name: secret-1401
          namespace: admin1401
        spec:
          volumes:
          - name: secret-volume
            secret:
              secretName: dotfile-secret
          containers:
          - command:
            - sleep
            - "4800"
            image: busybox
            name: secret-admin
            volumeMounts:
            - name: secret-volume
              readOnly: true
              mountPath: /etc/secret-volume
        ```

    1. And create the pod

        ```bash
        kubectl create -f admin.yaml
        ```

  </details>