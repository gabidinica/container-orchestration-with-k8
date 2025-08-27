# Demo Project

## Create Helm Chart for Microservices

### Technologies Used
- Kubernetes  
- Helm  

### Project Description
Create **1 shared Helm Chart** for all microservices, to reuse common **Deployment** and **Service** configurations for the services.

--- 

### Steps

1. **Create a new folder**
```bash
mkdir helm-chart-for-microservices
cd helm-chart-for-microservices
```

2. Go inside the project folder.
3. Add a new file named `config.yaml`.
4. In `config.yaml`, copy and paste the content from:  
   [config.yaml on GitHub](https://github.com/gabidinica/container-orchestration-with-k8/blob/main/microservices/config.yaml)
5. Open the project in **Visual Studio Code** (or any editor).
6. In the terminal, run:
```bash
helm create microservice
```

> A new folder named `microservice/` will be created with the Helm chart structure.

7. Go to the `templates/` folder inside the Helm chart and remove all files except: `deployment.yaml` and `service.yaml`
8. Open the `deployment.yaml` and `service.yaml` files inside the `templates/` folder and **remove all content**, leaving them empty.
9. Open the `values.yaml` file in the root of the Helm chart (`microservice/`) and **remove all content** as well.
10. Now the Helm chart is in a **clean state**, ready to start fresh.

## Basic Deployment Template

Inside `deployment.yaml` file paste the following content:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.varName }}
spec:
  selector:
    replicas: {{ .Values.appReplicas }}
    matchLabels: 
      app: {{ .Values.varName }}
  template:
    metadata:
      labels: 
        app: {{ .Values.varName }}
    spec:
      containers:
      - name: {{ .Values.varName }}
        image: "{{ .Values.appImage }}:{{ .Values.appVersion }}"
        ports: 
        - containerPort: {{ .Values.containerPort }}
```

And for the service:
```yaml
apiVersion: v1
kind: Service
metadata: 
  name: {{ .Values.varName }}
spec: 
  type: {{ .Values.containerType }}
  selector:
    app: {{ .Values.varName }}
  ports:
  - protocol: TCP
    port: {{ .Values.servicePort }}
    targetPort: {{ .Values.containerPort }}
```

> To set the variables in template file, we use the syntax: {{.Values.varName}}, where varName is the name we set.Values will come from values.yaml file once it will contain data in our case.
> For service type, we can make it configurable, because we have 1 microservice that is frontend and have type LoadBalancer and not clusterIp.
> In a Kubernetes Deployment, the `image` field has **two parts**:
> 1. **Repository and Image Name** – e.g., `nginx`, `myrepo/myapp`
> 2. **Image Tag / Version** – e.g., `1.21`, `v2.0.0`
> To set both parts dynamically using `values.yaml`, we can write: `"{{ .Values.appImage }}:{{ .Values.appVersion }}"`
> Number of replicas is also configurable, if it’s not mentioned then by default is 1.

## Dynamic Environment Variable

In a Helm `deployment.yaml`, dynamic environment variables can be added like this:
```yaml
env:
  - name: {{ .Values.containerEnvVar.name }}
    value: {{ .Values.containerEnvVar.value }}
```

> This example shows the case for a single environment variable.

To define multiple environment variables in a Helm `deployment.yaml`, we can use `range`:

```yaml
env:
  {{- range .Values.containerEnvVars }}
  - name: {{ .name }}
    value: {{ .value | quote }}
  {{- end }}
```

> .Values.containerEnvVars should be a list of environment variables defined in values.yaml.
> range iterates over each item in the list.
> quote ensures that the value is treated as a string.

## Setting Values for Helm Chart

 Using `values.yaml`

- The `values.yaml` file represents the **default values** for a Helm chart.  
- To customize for a specific microservice, we create a **custom values file** outside the `microservice/` folder.  
  - Example: `email-service-values.yaml`

### Steps

1. Copy all content from the default `values.yaml` file.  
2. Create a new file named `email-service-values.yaml`.  
3. Take the relevant values for the **email service** from `config.yaml`.  
4. Paste them correctly into `email-service-values.yaml`.  

- This allows Helm to deploy the email service using **customized values** without modifying the original chart.

To verify that the Helm chart is working with your custom values:
```bash
helm template -f email-service-values.yaml microservice
```

> This command renders the templates and shows the resulting Kubernetes manifests without deploying them.

To check for possible issues in the Helm chart, use:
```bash
helm lint -f email-service-values.yaml microservice
```

## Deploy services into cluster

| Helm Command | `-f email-service-values.yaml` | `emailservice` | `microservice` |
|--------------|-------------------------------|----------------|----------------|
| `helm install` | Override values from a file | Release name | Chart name |

1. **Install the Helm chart with custom values:**

```bash
helm install -f email-service-values.yaml emailservice microservice
```

2. Check Helm releases: `helm ls`
3. Check Kubernetes pods: kubectl get pod

### Create Values Directory for Microservices

1. **Create a new directory to store values files:**
```bash
mkdir values
```

2. Create values files for each microservice:
- For each microservice, create a separate file (e.g., email-service-values.yaml).
- Copy the default values from values.yaml.
- Replace them with the correct values for each microservice from config.yaml.

3. Organize the files: Place all microservice values files inside the values/ directory.
4. After moving the microservice values files into the `values/` folder, run the Helm install command pointing to the specific file:
```bash
helm install -f values/email-service-values.yaml emailservice microservice
```

5. Check Helm releases: `helm ls`
6. Check Kubernetes pods: `kubectl get pod`

## Create Redis Helm Chart

1. **Organize chart folders:**
```bash
mkdir charts
mv microservice charts/
cd charts
```

2. Create a Redis Helm chart: 
```bash
helm create redis
```

3. Remove anything from `templates` folder that’s under `redis` folder. Remove also the content from `values.yaml` file.
4. Inside templates folder add deployment.yaml and service.yaml files. You can find the files content inside  charts *redis -> templates* folder in this repo.
5. **Go back to the charts folder:**
```bash
cd ..
```

6. Validate the Redis chart with custom values:
```bash
helm template -f values/redis-values.yaml charts/redis
```

7. Use the `--dry-run` option to simulate the installation and preview the generated manifests:

```bash
helm install --dry-run -f values/redis-values.yaml rediscart charts/redis
```

# Install All Services with Helm

```bash
helm install -f values/redis-values.yaml rediscart charts/redis

helm install -f values/email-service-values.yaml emailservice charts/microservice
helm install -f values/cart-service-values.yaml cartservice charts/microservice
helm install -f values/currency-service-values.yaml currencyservice charts/microservice
helm install -f values/payment-service-values.yaml paymentservice charts/microservice
helm install -f values/recommendation-service-values.yaml recommendationservice charts/microservice
helm install -f values/productcatalog-service-values.yaml productcatalogservice charts/microservice
helm install -f values/shipping-service-values.yaml shippingservice charts/microservice
helm install -f values/ad-service-values.yaml adservice charts/microservice
helm install -f values/checkout-service-values.yaml checkoutservice charts/microservice
helm install -f values/frontend-values.yaml frontendservice charts/microservice
```

