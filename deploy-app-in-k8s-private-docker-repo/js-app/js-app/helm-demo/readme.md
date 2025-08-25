# Demo Project

## Install a Stateful Service (MongoDB) on Kubernetes using Helm

### Technologies Used
- Kubernetes (K8s)
- Helm
- MongoDB
- Mongo Express
- Linode LKE
- Linux

### Project Description
- [Create a managed K8s cluster with Linode Kubernetes Engine (LKE)](#create-a-managed-k8s-cluster-with-linode-kubernetes-engine-lke)  
- [Deploy a replicated MongoDB service in the LKE cluster using a Helm chart](#deploy-a-replicated-mongodb-service-in-the-lke-cluster-using-a-helm-chart)  
- [Configure data persistence for MongoDB with Linode’s cloud storage](#configure-data-persistence-for-mongodb-with-linodes-cloud-storage)  
- [Deploy UI client Mongo Express for MongoDB](#deploy-ui-client-mongo-express-for-mongodb)  
- [Deploy and configure nginx ingress to access the UI application from the browser](#deploy-and-configure-nginx-ingress-to-access-the-ui-application-from-the-browser)

---

## Create a managed K8s cluster with Linode Kubernetes Engine (LKE)

1. Create an account on [Linode](https://cloud.linode.com) if you don’t have one already.  
2. After you login, click on the **Create** button and choose **Kubernetes**.  
3. On the **Create cluster** view, configure the following:  
   - **Cluster label:** test  
   - **Cluster Tier:** LKE  
   - **Region:** Frankfurt (eu-central)  
   - **Kubernetes version:** 1.33  
   - **Akamai App Platform:** No  
   - **HA Control Plane:** No  
4. On **Add Node Pools**, click on the **Shared CPU** tab and choose **Linode 4GB**:  
   - Select **2** nodes  
   - Click on **Add** button  
5. After these steps, click on the **Create cluster** button.

### Accessing the Linode Cluster from Your Laptop

1. After the pods are running, download the `test-kubeconfig.yaml` file from **Kubeconfig**.  
   - This file contains the credentials and certificates for your cluster.
2. Move the downloaded file from your **Downloads** folder to your working folder (e.g., `demo-helm`).
3. Open a terminal, navigate to your working folder, and set the file permissions so that only your user can read it:  
```bash
cd demo-helm
chmod 400 test-kubeconfig.yaml
```

4.Set the `KUBECONFIG` environment variable to use your downloaded kubeconfig file:  
```bash
export KUBECONFIG=test-kubeconfig.yaml
```

5.Verify access to your cluster by listing the nodes: `kubectl get nodes`
> You should see the 2 Linode nodes in the output.

## Deploy a replicated MongoDB service in the LKE cluster using a Helm chart

### Prerequisites

1. Install **Helm**: [Helm Installation Guide](https://helm.sh/docs/intro/install/)  
2. The Bitnami Helm chart repository information can be found here:  
   [Bitnami MongoDB Helm Chart](https://github.com/bitnami/charts/tree/main/bitnami/mongodb)  

---

1. Since the MongoDB chart is maintained by Bitnami, add the repository:  
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

2. To view all charts in the Bitnami repository: 
```bash
helm search repo bitnami
```

3. To specifically search for the MongoDB chart:
```bash
helm search repo bitnami/mongodb
```

> We’re gonna overwrite some values that are already defined in chart by creating a file that will overwrite. The file is helm-mongoldb.yaml that contains the values.

## Configure data persistence for MongoDB with Linode’s cloud storage

### Deploy MongoDB with Custom Values

1. For storage, we will use `linode-block-storage` to provide a **persistent volume** for MongoDB.  
2. Execute the Helm install command to deploy MongoDB using your custom values:  
```bash
helm install mongodb --values helm-mongodb.yaml bitnami/mongodb
```

- **mongodb**: The name you are giving to the MongoDB StatefulSet.
- **helm-mongodb.yaml**: The values file containing your custom overrides.
- **bitnami/mongodb**: The Helm chart name.

3. Check the pods:  `kubectl get pod`
4. See all resources created in the namespace: `kubectl get all`
5. View the secrets, which contain the root password and credentials defined in your `helm-mongodb.yaml` file:
```bash
kubectl get secret
```

6. To verify persistent volumes:
- Open your Linode dashboard in the browser.
- Go to Volumes.
- You should see 3 volumes corresponding to the MongoDB pods.

## Deploy UI client Mongo Express for MongoDB

1. The YAML file for deploying Mongo Express is `helm-mongo-express.yaml`.  
2. To check the secret for MongoDB:  
```bash
kubectl get secret mongodb -o yaml
```

> You will see the mongodb-root-password and other credentials used in your deployment.

- The environment variable ME_CONFIG_MONGODB_URL is formed as follows:
> protocol://username:password@pod-name:port

- protocol: mongodb
- username & password: Credentials from the secret
- pod-name: e.g., mongodb-0 (refers to the first pod in the StatefulSet)
- port: Default MongoDB port (usually 27017) 
- Headless Service:
> Used when you want to communicate directly with a specific pod in the StatefulSet.In this case, it allows Mongo Express to connect to mongodb-0 directly.

-  The service is **internal** by default.  

3. To create the service in the cluster, apply the YAML file:  
```bash
kubectl apply -f helm-mongo-express.yaml
```

4. Check the pods to ensure Mongo Express is running: `kubectl get pod`
5. View the logs of the Mongo Express pod (replace the pod name with your actual pod name):
```bash
kubectl logs mongo-express-name
```

## Deploy and configure nginx ingress to access the UI application from the browser

### Deploy Nginx Ingress Controller using Helm

1. **Add the Helm chart repository** for the Ingress controller:
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```

2. Install the Nginx Ingress controller using Helm:
```bash
helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.publishService.enabled=true
```

- **nginx-ingress**: The name you are giving to this Helm release.  
- **ingress-nginx/ingress-nginx**: The Helm chart for the Nginx Ingress controller.  
- `--set controller.publishService.enabled=true`: Ensures the controller publishes its service so it can be accessed externally.

3. **Check the Ingress controller pod(s):**  
```bash
kubectl get pod
```

4. Open the **Linode dashboard** in your browser.  
5. Go to **NodeBalancer** to find the external IP address assigned to the Ingress controller.
6. Ingress rules can be found in `helm-ingress.yaml`.
7.  **Set the Host:**  
- The host must be a valid domain address.  
- Replace it with the hostname of the Linode NodeBalancer.  
- Example: `Host Name: EXTERNAL_IP.ip.linodeusercontent.com`  
8. **Apply the ingress configuration to the cluster:**  
```bash
kubectl apply -f helm-ingress.yaml
```

9. Verify the ingress resource: `kubectl get ingress`
10. Access Mongo Express UI:
- Copy the domain name from the ingress output.
- Paste it into your browser to open the Mongo Express interface.

### Check Persistence by Scaling MongoDB Replicas

1. **Scale down the MongoDB StatefulSet to 0 replicas:**  
```bash
kubectl scale --replicas=0 statefulset/mongo
```

2. Verify that all pods are terminated: `kubectl get pod`
3. Scale the MongoDB StatefulSet back up to 3 replicas:
```bash
kubectl scale --replicas=3 statefulset/mongo
```

