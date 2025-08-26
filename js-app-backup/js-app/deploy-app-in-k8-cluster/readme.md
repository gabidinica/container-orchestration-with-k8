# Demo Project

## Project Overview
Deploy MongoDB and Mongo Express into a local Kubernetes cluster.

## Technologies Used
- Kubernetes
- Docker
- MongoDB
- Mongo Express

## Project Description
- [Setup local K8s cluster with Minikube](#setup-local-k8s-cluster-with-minikube)  
- [Deploy MongoDB and Mongo Express with configuration and credentials extracted into ConfigMap and Secret](#deploy-mongodb-and-mongo-express-with-configuration-and-credentials-extracted-into-configmap-and-secret)

---

### Setup local K8s cluster with Minikube

- **Install Docker:** [Docker Get Started](https://www.docker.com/get-started/)  
- **Install Minikube:** [Minikube Installation](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Farm64%2Fstable%2Fhomebrew)  
  > You’ll need to select your architecture. On macOS, it can be installed using Homebrew.  

> Note: Docker must also be installed locally, as we’ll use Docker to run Minikube as a Docker container.

**Starting Minikube**
```bash
minikube start --driver=docker
```

**Checking Minikube Status**
```bash
minikube status
```

**Checking Kubernetes Nodes**
```bash
kubectl get nodes
```

### Deploy MongoDB and Mongo Express with configuration and credentials extracted into ConfigMap and Secret

To check all the components that are installed in the cluster, run:
```bash
kubectl get all
```

> You can check the details for MongoDB on Docker Hub: [MongoDB Docker Hub](https://hub.docker.com/_/mongo)
The **DB User** and **DB Password** are referenced in a **Kubernetes Secret** file, not in the YAML manifests, as secrets are stored securely in Kubernetes, not in the repository.

#### MongoDB yaml file

The deployment file for MongoDB can be found in the `mongo.yaml` file. It currently has only **one replica**.

#### Creating MongoDB Secret

Create a new file `mongo-secret.yaml` for the secret with the following configuration:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
type: Opaque
data:
  mongo-root-username: {paste-username-encoded-value}
  mongo-root-password: {paste-password-encoded-value}
```

> Replace `{paste-username-encoded-value}` and `{paste-password-encoded-value}` with your base64-encoded MongoDB username and password. `Opaque` is the default type for arbitrary key-value pairs.

To create your *secret values*, go to the terminal and base64 encode them:
```bash
echo -n 'username' | base64
```

> Copy the resulting value into the mongo-secret.yaml file for `mongo-root-username`.

```bash
echo -n 'password' | base64
```

> Copy the resulting value into the mongo-secret.yaml file for `mongo-root-password`.

#### Creating the Secret Before Deployment

Secrets must always be created **before** the deployment, as they are referenced in the `mongo.yaml` file.

To create the secret based on your configuration file, navigate in the terminal to the folder where you added the config file.  
For example:

```bash
cd k8/deploy-app-in-k8-cluster
ls
kubectl apply -f mongo-secret.yaml 
```

#### Referencing the Secret in the Deployment

After the secret is created, it can be referenced in the deployment configuration (`mongo.yaml`) like this:

```yaml
env:
  - name: MONGO_INITDB_ROOT_USERNAME
    valueFrom:
      secretKeyRef: 
        name: mongodb-secret
        key: mongo-root-username
  - name: MONGO_INITDB_ROOT_PASSWORD
    valueFrom:
      secretKeyRef:
        name: mongodb-secret
        key: mongo-root-password
```

Apply the MongoDB deployment changes:
```bash
kubectl apply -f mongo.yaml
```

Check all resources in the cluster: `kubectl get all`
Check the status of the pods: `kubectl get pod`
Check detailed information about a specific pod: `kubectl describe pod pod_name`

#### Creating an Internal Service for MongoDB

We’ll create an **internal service** so that other components and pods can communicate with MongoDB.  
The **Deployment** and **Service** should be in the same `mongo.yaml` file because they belong together.

Add the following to your `mongo.yaml` file:

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
spec:
  selector: 
    app: mongodb
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
```

Apply the MongoDB deployment and service changes:
```bash
kubectl apply -f mongo.yaml
```

Check that the service was created: `kubectl get service`
Get detailed information about the service: `kubectl describe service mongodb-service`
Check that the IP address and port are correct: `kubectl get pod -o wide`
Check all MongoDB-related components: `kubectl get all | grep mongodb`

#### Mongo Express Deployment and Service

- You can check the Mongo Express image on Docker Hub: [Mongo Express Docker Hub](https://hub.docker.com/_/mongo-express)  
- A new file `mongo-express.yaml` has been added with the configuration (check the repository).  
- Mongo Express starts at **port 8081**.  

#### Database URL Configuration
- For `DB_URL`, refer to the GitHub repository for the latest configuration: [Mongo Express GitHub](https://github.com/mongo-express/mongo-express)  
- The **username** and **password** come from the previously created secret.  
- The value of `ME_CONFIG_MONGODB_URL` will use a **ConfigMap** so other components can reference it and the configuration is centralized.

#### Creating a ConfigMap for MongoDB

Create a new file `mongo-configmap.yaml` with the following configuration:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
data:
  database_url: "mongodb-service:27017"
```

#### Referencing the ConfigMap in `mongo-express.yaml`

To reference the `database_url` from the ConfigMap, similar to how secrets are referenced, add the following to your `mongo-express.yaml` file:

```yaml
- name: DATABASE_URL
  valueFrom:
    configMapKeyRef:
      name: mongodb-configmap
      key: database_url
- name: ME_CONFIG_MONGODB_URL
  value: "mongodb://$(ME_CONFIG_MONGODB_ADMINUSERNAME):$(ME_CONFIG_MONGODB_ADMINPASSWORD)@$(DATABASE_URL)"
```

Order of execution matters, so apply the ConfigMap **first**:
```bash
kubectl apply -f mongo-configmap.yaml
```

Then apply the Mongo Express deployment:
```bash
kubectl apply -f mongo-express.yaml
```

Check pod's status: `kubectl get pod`
Check the logs for the Mongo Express pod: `kubectl logs mongo-express-name`

#### Creating an External Service for Mongo Express

Add a service in the same `mongo-express.yaml` file to expose Mongo Express externally.  
Use a **NodePort** so it can be accessed via an external IP address (port must be between 30000-32767).

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service
spec:
  selector: 
    app: mongo-express
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
      nodePort: 30000
```

Check all services in the cluster:
```bash
kubectl get service
```

> The mongo-express-service will have the *LoadBalancer* type.
> The default type is *ClusterIP*, which gives the service a cluster-internal IP, making it reachable *only within the cluster*

To assign a public IP and access the service externally:
```bash
minikube service mongo-express-service
```

> After executing this command, your browser will open, and you’ll see the Mongo Express interface.

### Optional: Disable Basic Authentication in Mongo Express

> OBS: If you are not automatically logged in, update `mongo-express.yaml`.

After the following line:

```yaml
- name: ME_CONFIG_MONGODB_URL
```

add:
```yaml
- name: ME_CONFIG_BASICAUTH
  value: "false"
```

Then reapply the changes and rerun the Minikube command:
```bash
kubectl apply -f mongo-express.yaml
minikube service mongo-express-service
```

> Your browser should open and display the Mongo Express interface without authentication.
