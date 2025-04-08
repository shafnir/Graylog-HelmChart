<p align="center">
  <img src="https://github.com/user-attachments/assets/ce37e878-2eb9-44f4-9817-99796915682c?s=800&v=4" alt="Graylog Logo" width="450"/>
</p>

<h1 align="center">Graylog Open Helm Chart</h1>

<p align="center">
  A fully working Helm Chart of <strong>Graylog Open Source.</strong>
</p>



## 📌 Overview

This setup deploys a minimal Graylog stack with:

- **Graylog Core**
- **Graylog DataNode**
- **Secrets Management**
- **Persistent Volumes (PVCs)** via `hostpath-provisioner`
- **NodePort Exposure**

## Additional Notes

Please go through all the values in values.yaml and change according to your needs.  
DO NOT DEPLOY the chart before populating the first 2 values (graylogPasswordSecret and graylogRootPasswordSHA256), it will not work without it!  
Please review the allocated resources in the PVCs and the mongodb StatefulSet and ensure it matches your requirements.  
This deployment is made for a small lab environment, it is recommened to increase the replicas, the allocated storage in the PVCs and the resource requests and limits of the datanode.


## ⚙️ Prerequisites

- Kubernetes cluster (e.g. Minikube or real cluster)  
- `kubectl` CLI configured
- `helm` installed
- `hostpath-provisioner` installed from [ArtifactHub](https://artifacthub.io/packages/helm/rimusz/hostpath-provisioner)



## 🚀 Deployment Steps

1. Generate the graylogPasswordSecret for the values.yaml file, with the following command:

    ```bash
    < /dev/urandom tr -dc A-Z-a-z-0-9 | head -c${1:-96}; echo
    ```

2. Generate the graylogRootPasswordSHA256 for the values.yaml file, with the following command:

    ```bash
    echo -n "Enter Password: " && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1
    ```

3. After filling the values in the values.yaml, install the chart:

    ```bash
    helm repo add graylog https://shafnir.github.io/Graylog-HelmChart/graylog
    helm install graylog -n graylog graylog/graylog --version 0.1.0 --create-namespace
    ```

4. Verify that pods are running:

    ```bash
    kubectl get pods -n graylog
    ```

    Example output:
    ```bash
    NAME                        READY   STATUS    RESTARTS   AGE
    datanode-5dcff9cffb-qf26r   1/1     Running   0          57m
    graylog-74558bdf5b-zcc8h    1/1     Running   0          61m
    mongodb-0                   1/1     Running   0          61m
    ```

5. Retrieve the initial admin password from Graylog logs (based on the graylog pod name):

    ```bash
    kubectl logs -n graylog graylog-74558bdf5b-zcc8h
    ```

    You should see something like:

    ```bash
    Initial configuration is accessible at 0.0.0.0:9000, with username 'admin' and password 'bhQRFNUvIe'.
    Try clicking on http://admin:bhQRFNUvIe@0.0.0.0:9000
    ```

6. Access the Graylog UI:

    ```
    http://[your-node-ip]:30900
    ```

7. After completing the initial setup wizard, log in with username `admin` and the password you set in **Step 3**.

--- 

8. To uninstall the chart, use this command:

   ```bash
   helm uninstall -n graylog graylog
   ```
9. To delete all of the related PV and PVCs, use the following command:

    ```bash
    kubectl delete -n graylog pv,pvc --all
    ```
   

---
