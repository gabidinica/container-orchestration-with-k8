# Demo Project

## Deploy Microservices with Helmfile

### Technologies Used
- Kubernetes
- Helm
- Helmfile

### Project Description
- Deploy Microservices with Helm  
- Deploy Microservices with Helmfile

----

## Deploy Microservices with Helm  

1. **Change permissions to make the script executable:**

```bash
chmod u+x install.sh
```

2. Run the script:
```bash
./install.sh
```

3. Check Kubernetes Pods
```bash
kubectl get pod
```

4. **Make the uninstall script executable:**
```bash
chmod u+x uninstall.sh
```

5. Run the uninstall script:
```bash
./uninstall.sh
```

## Deploy Microservices with Helmfile

1. **Use a `helmfile.yaml`**
- The `helmfile.yaml` contains the deployment configuration for all microservices.  
- Its content can be found in the attached repository.  
- You can overwrite values in the `helmfile.yaml`, similar to how we did for Redis with custom values files.

2. **Install the Helmfile tool:**
```bash
brew install helmfile
```

3.  **Check that the cluster is clean:**
```bash
kubectl get pod
```

4.Update /sync cluster with what we’ve declared in our *helmfile*:
```bash
helmfile sync
```

5. To see all releases deployed by Helmfile:
```bash
helmfile list
```

6. Check Kubernetes Pods
```bash
kubectl get pods
```

- If you are using Minikube, you can open the frontend service in your browser:
```bash
minikube service frontend
```

- For Linode, **Get the LoadBalancer IP** and paste it into the browser. Website should load successfully.

7. **Destroy all releases managed by Helmfile:**
```bash
helmfile destroy
```

8. Verify Helm releases are removed: `helm ls`
9. Check Kubernetes pods to ensure cleanup:
```bash
kubectl get pod
```

