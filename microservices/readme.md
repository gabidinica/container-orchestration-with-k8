# Demo Project  
**Deploy Microservices application in Kubernetes with Production & Security Best Practices**

---

## Technologies Used
- Kubernetes  
- Redis  
- Linux  
- Linode LKE  

---

## Project Description
- Create K8s manifests for **Deployments** and **Services** for all microservices of an online shop application  
- Deploy microservices to **Linode’s managed Kubernetes cluster**

---- 

The project we’re gonna use is forked/cloned from Google, the project can be found here:  
👉 [Microservices Demo Repository](https://github.com/techworld-with-nana/microservices-demo)  
or already in this folder.

---
  
## Create **Kubernetes manifests** for **Deployments** and **Services** for all microservices of an online shop application.

#### Note  
I’ve added a new file named **`config.yaml`** and to make it easier to start, it has only **one Deployment** and **one Service** with the **minimum information**.
But the whole config file it's completed and you can find it above.

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: xxx
spec:
  selector:
    matchLabels: 
      app: xxx
  template:
    metadata:
      labels: 
        app: xxxx
    spec:
      containers:
      - name: xxx
        image: xxx
        ports: 
        - containerPort: xxxx
---
apiVersion: v1
kind: Service
metadata: 
  name: xxx
spec: 
  type: ClusterIP
  selector:
    app: xxxx
  ports:
  - protocol: TCP
    port: xxxx
    targetPort: xxxx
```

We’re gonna need **each one of these manifests** for **each microservice**.  
Since we’re deploying **10 microservices + Redis**, we’re going to have this exact configuration **11 times** in the `config.yaml` file.  

---

### Microservices List
1. **EmailService**  
2. **RecommendationService** (which talks with **ProductCatalogueService**)  
3. **ProductCatalogueService**  
4. **PaymentService**  
5. **CurrencyService**  
6. **ShippingService**  
7. **AdService**  
8. **CartService** (connected to **Redis** to save shopping cart information)  
9. **Redis**  
10. **CheckoutService** (which talks to many other services)  
11. **Frontend**

### Special Considerations

#### Redis
- We will attach a **volume** in order to **persist data**.  
- For simplicity, we’ll mount an **empty directory** called `redis-data` inside the pod.  

#### Frontend
- The **Frontend microservice** needs the **endpoints of all the microservices** that it communicates with.  
- It has to be **externally accessible**, so in this case the **Service type** will be `NodePort`.  

## Deploy Microservices to Linode’s Managed Kubernetes Cluster

### Step 1: Create a Linode Account
- Go to [Linode Cloud](https://cloud.linode.com) and create an account if you don’t already have one.  
- Log in to your account.  

### Step 2: Create a Kubernetes Cluster
1. Click on the **Create** button and select **Kubernetes**.  
2. In the **Create Cluster** view, configure the following:  
   - **Cluster label**: `online-shop-microservices`  
   - **Cluster Tier**: `LKE`  
   - **Region**: `Frankfurt (eu-central)`  
   - **Kubernetes version**: `1.33`  
   - **Akamai App Platform**: `No`  
   - **HA Control Plane**: `No`  

### Step 3: Configure Node Pools
1. Under **Add Node Pools**, click on the **Shared CPU** tab.  
2. Choose **Linode 4GB**.  
3. Select **2 nodes**.  
4. Click **Add**.  

### Step 4: Finalize
- Click the **Create Cluster** button.  
- Wait for the cluster to be provisioned.  

### - Download Kubeconfig
- After the pods are running, go to **Kubeconfig** in the Linode dashboard.  
- Download the file:  
  **`online-shop-microservices-kubeconfig.yaml`**  
  (This file contains the credentials and certificates for your cluster).  

### - Move the Kubeconfig File
- Move the downloaded file from your **Downloads** folder to your working folder (for example, `demo-helm`):  
```bash
mv ~/Downloads/online-shop-microservices-kubeconfig.yaml ~/microservices/
```

### - Restrict file permissions so that only your user can read it:
```bash
chmod 400 online-shop-microservices-kubeconfig.yaml
```

Step 1: Point kubectl to the Linode Cluster
```bash
export KUBECONFIG=~/demo-helm/online-shop-microservices-kubeconfig.yaml
```

Step 2: Verify cluster access: `kubectl get nodes`

Step 3: Create a namespace:
```bash
kubectl create ns microservices
```

Step 4: Deploy yaml file:
```bash
kubectl apply -f config.yaml -n microservices
```

Step 5: Check the pods: `kubectl get pods -n microservices`

Step 6: Check the services: `kubectl get svc -n microservices`
