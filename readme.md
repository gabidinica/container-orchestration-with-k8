# Demo Project  
**Deploy Mosquitto message broker with ConfigMap and Secret Volume Types**

## Technologies used
- Kubernetes  
- Docker  
- Mosquitto  

## Project Description
Define configuration and passwords for Mosquitto message broker with ConfigMap and Secret Volume types.

### ConfigMap & Secret files

- In **config-file.yaml**, we have the file named **mosquitto.conf** and, after the pipe (`|`), is the content inside it.  

- In **secret-file**, we have a file named **secret.file** and, similar to the config-file, after the pipe (`|`) is the content of it **encoded in Base64**.  

### 🔑 Encode Text in Base64
To encode a text in Base64, run in the terminal:
```bash
echo -n 'text to be encoded' | base64
```

🔐 Client Certificates

For the case of client certificates, an example of the YAML file can be found in **example-secret-certificate.yaml**, which is similar to the secret file.

### Running Mosquitto Without Volumes

In **mosquitto-without-volumes.yaml**, we have the default Mosquitto container without the volumes attached.  
Navigate to the folder in the terminal, to apply the changes and run:
```bash
kubectl apply -f mosquitto-without-volumes.yaml
```

Check the pod: 
```bash
kubectl get pod
```

🔑 Access the Mosquitto Pod:
```bash
kubectl exec -it pod-name -- /bin/sh
```

Once inside the container:
1. List files: `ls`
2. Go to Mosquitto folder: `cd mosquitto`
3. Go to config folder: `cd config`
4. List the contents of the folder: `ls`
5. Exit the pod now: `exit`

🗑️ Delete the Deployment: 
```bash
kubectl delete -f mosquitto-without-volumes.yaml
```

### 🔄 Overwrite Mosquitto Config

Now we’ll overwrite the Mosquitto config file that we saw earlier with the one that has the ConfigMap and Secret mapped.

Before doing this, we need to map the config-file.yaml and secret-file.yaml:
```bash
kubectl apply -f config-file.yaml
kubectl apply -f secret-file.yaml
```

To check these: 
```bash
kubectl get configmap
kubectl get secret
```

#### 📦 Mosquitto Deployment with Volumes

The file **mosquitto.yaml** contains the volumes mapped.  

Initially, we mount them at the **pod level**, under `volumes` (on the same level as `containers`):

```yaml
volumes:
  - name: mosquitto-config
    configMap: 
      name: mosquitto-config-file
  - name: mosquitto-secret
    secret:
      secretName: mosquitto-secret-file
```

After defining the volumes, we need to **mount them inside the container** using `volumeMounts`:

```yaml
volumeMounts:
  - name: mosquitto-config
    mountPath: /mosquitto/config
  - name: mosquitto-secret
    mountPath: /mosquitto/secret
    readOnly: true
```

📝 Notes
- `mountPath` references the path in the container’s filesystem, as we’ve seen earlier in `/mosquitto/config`.
- The `secret` folder does not exist by default, so it will be automatically created at `/mosquitto/secret`.
- `readOnly` attribute ensures the data is only read, not modified. This is especially important for certificates, which only need to be read.

1. Apply changes:
```bash
kubectl apply -f mosquitto.yaml
```

2. Check the pod: `kubectl get pod`
3. Access the Mosquitto Container: `kubectl exec -it pod-name -- /bin/sh`

Once inside the container:
1. List files: `ls`
2. Go to Mosquitto folder: `cd mosquitto`
3. List contents: `ls`
> 👉 You’ll see, for example, the secret folder, as initially it wasn't there.
4. Check the config content to see that it was overwritten:
```bash
cd config
cat mosquitto.conf
```

