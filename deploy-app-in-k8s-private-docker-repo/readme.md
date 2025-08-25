# Demo Project

Deploy our web application in a Kubernetes (K8s) cluster from a private Docker registry.

## Technologies Used
- Kubernetes
- Helm
- AWS ECR
- Docker

## Project Description
1. [Create a Secret for credentials for the private Docker registry](#create-secret-for-credentials).2. [Configure the Docker registry secret in the application Deployment component](#configure-docker-registry-secret).  
3. [Deploy the web application image from our private Docker registry in the K8s cluster](#deploy-web-application).

---

## Create Secret for credentials

To create a secret for your private Docker registry:

- You can check the parameters for `docker login` in the official documentation: [Docker Login CLI Reference](https://docs.docker.com/reference/cli/docker/login/).  
- If you’re using **Amazon ECR**, go to the **View Push Commands** section, and copy the first login command. This command gets the login password into stdin, which you can then use to create the Kubernetes secret.

### Further I will use my DockerHub credentials to deploy the web app.

1. Enter password securely:
```bash
read -s DOCKER_PWD
```

2. Create the secret using the password you just typed:
```bash
kubectl create secret docker-registry regcred \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=username \
  --docker-password="$DOCKER_PWD" \
  --docker-email=email-address
```

3. In you terminal type: `docker login`
Now you are logged into Docker. To check the authentication type, run:
```bash
cat ~/.docker/config.json
```

You should see an entry like: `"credsStore": "desktop"`

4. Open a new tab in your terminal and **SSH into Minikube:**
```bash
minikube ssh
```

5. Check the current path: `ls`
6. See that there’s no Docker folder initially: `ls -a`
7. Login to Docker Hub: 
```bash
docker login -u <dockerhub-username>
```

- You will be prompted to type your password.
- If the login is successful, you should see: Login Succeeded

8. Verify that the .docker folder is created: `ls -a`
9. Check the config.json file inside .docker: 
```bash
cat .docker/config.json
```

## Configure Docker Registry Secret


To copy the `config.json` file from Minikube to your local machine, run:
```bash
minikube cp minikube:/home/docker/.docker/config.json /Users/<you-folder-name>/.docker/config.json
```

> This command copies the Docker configuration from the Minikube VM to your local .docker folder.

After copying the file, your local Docker CLI will use the same configuration and credentials as the Docker daemon inside Minikube.

To check the file content:
```bash
cat ~/.docker/config.json
```

> You should see the Docker credentials and settings that were copied from Minikube.

The `.dockerconfigjson` file should be **base64 encoded** before using it in a Kubernetes secret.

1. Encode the Docker config file:
```bash
cat ~/.docker/config.json | base64
```

2. Copy the output value from the terminal.
3. Paste the base64-encoded value into your docker-secret.yaml under the data section:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-registry-key
data:
  .dockerconfigjson: <encoded-password-from-terminal>
type: kubernetes.io/dockerconfigjson
```

Once your `docker-secret.yaml` is ready, apply it to your Kubernetes cluster:
```bash
kubectl apply -f docker-secret.yaml
```

To check the secrets:
```bash
kubectl get secret
```

View the full details of your secrets in YAML format:
```bash
kubectl get secret -o yaml
```

### Alternative: Create Docker Secret in One Step

You can create the secret directly using your Docker login credentials in a single command:
```bash
kubectl create secret docker-registry my-registry-key-two \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=user \
  --docker-password=pwd
```

> Replace user and pwd with your Docker Hub username and password.

Verify the secret: `kubectl get secret`
> You should see both registry secrets listed, including the newly created `my-registry-key-two`.

## Deploy Web Application

### Pre-requisites: Build and Push Docker Image to Docker hub

1. **Build the Docker image:**
```bash
docker build -t my-app-1 .
```

2. Tag the Docker image for your repository:
```bash
docker tag my-app-1 <docker_repo>:my-app-1
```

3. Push the image to Docker Hub:
```bash
docker push <docker_repo>:my-app-1
```

> After this, your image will be available in your Docker Hub repository for use in Kubernetes deployments.

Check with: `docker images`

You’ll find the `my-app-deployment.yaml` file inside the `js-app` folder.

- Under the **pod configuration**, you need to specify the **complete name of the Docker image** you want to pull from Docker Hub.  
- Since this example uses a Docker Hub image, the image section in the YAML should be:

```yaml
image: <repo-name>:my-app-1
```

> Replace <repo-name> with your Docker Hub repository name.

```yaml
spec:
  imagePullSecrets:
    - name: my-registry-key
  containers:
    - name: my-app
      image: <private-docker-repo>:my-app-1
      imagePullPolicy: Always
      ports:
        - containerPort: 3000
```

> imagePullPolicy: Always -> Forces Kubernetes to always pull the image from the Docker repository, even if the image exists locally on the node.

Apply the deployment YAML file:
```bash
kubectl apply -f my-app-deployment.yaml
```

To test that both secrets are working, create a new deployment:
- Rename the deployment to `my-app-2` everywhere in the YAML.
- Update imagePullSecrets to use the second secret:
```bash
spec:
  imagePullSecrets:
    - name: my-registry-key-two
```

Apply the updated deployment:
```bash
kubectl apply -f my-app-deployment.yaml
```

Check the status of all pods:
```bash
kubectl get pods
```

- You should see *both pods* (my-app-1 and my-app-2) listed.
- Ensure their *STATUS* column shows Running, indicating the deployments are successfully using the Docker registry secrets.

**⚠️ Observation:**  

If the secret wasn’t created correctly, you can reapply or update it using:

```bash
kubectl create secret docker-registry my-registry-key-two \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=dockerhub-username \
  --docker-password='dockerhub-pwd' \
  --docker-email=docker-email \
  --dry-run=client -o yaml | kubectl apply -f -
```

- Explanation:
	- `--dry-run=client -o yaml` generates the secret manifest without applying it immediately.
	- Piping it to `kubectl apply -f -` creates or updates the secret if it already exists.
