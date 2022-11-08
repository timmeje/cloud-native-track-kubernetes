# Lab 6 - Config Maps & Secrets

## Task 1: Creating a configmap

With a configmap we are able to easily pass environment variables to the 
deployment we are creating.

Create a file `container-info.env` with the following content:

```
# container-info.env

IMAGE_COLOR=green
```

This will pass an environment variable to the pods that we will create with our 
deployment. We are now able to create a configmap with this file.

```
kubectl create configmap container-info-env --from-env-file='./container-info.env'
```

Verify that the configmap has been created succesfully:

```
kubectl get configmap container-info-env

---

NAME                 DATA      AGE
container-info-env   1         59s
```

```
kubectl describe configmap container-info-env

---

Name:		container-info-env
Namespace:	...
Labels:		<none>
Annotations:	<none>

Data
====
IMAGE_COLOR:
----
green
Events:	<none>
```

## Task 2: Adding the configmap in the deployment.

The deployment we create now looks a lot like the deployment we created in the 
previous labs.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: container-info
  labels:
    app: container-info
spec:
  replicas: 1
  selector:
    matchLabels:
      app: container-info
  template:
    metadata:
      labels:
        app: container-info
    spec:
      containers:
      - name: container-info
        image: gluobe/container-info:variable
        ports:
        - containerPort: 80
        envFrom:
          - configMapRef:
              name: container-info-env
```     

Copy the content above into a file `lab-04-deployment.yml`.

Now we create the new deployment with our environment variables.

```
kubectl apply -f lab-04-deployment.yml

---

deployment "container-info" created
```

We will expose the deployment again.

```
kubectl expose deployment container-info --type=NodePort --name=container-info

---

service "container-info" exposed
```

Now you can try connecting to the service in you browser, as before.

## Task 3: Updating an existing configmap

Configmaps make it possible to change configuration of an application (pod), 
without needing to completely rebuild a container image.  Let's try this.

Issue the following command, and replace the image color to `yellow` and save 
the  file (you will get a `configmap "container-info-env" edited` message when 
succesful):

```
kubectl edit configmap container-info-env

---

configmap "container-info-env" edited
```

> If you're not used to command line text editors, you can also simply edit your file using an editor of your choice and re-apply the file.

* After making this change, reload the service in your browser. Did the colour change? If not, what can you do to make it change? 

## Task 4: Creating secrets

Secrets are handled slightly differently in a Kubernetes cluster. Like configmaps, there are 3 ways to
handle these secrets. From `literal`, from a `file` and declaratively from a `YAML` file.

If you want to create a secret from `literal` it looks like this.

```
kubectl create secret generic lab-06-secret --from-literal=password=N0T$oS3cREtP@SSw0rD

---

secret/lab-06-secret created
```

This secret can be listed by the command:

```
kubectl get secrets

---

NAME            TYPE                                  DATA   AGE
user-token      kubernetes.io/service-account-token   3      6m40s
lab-06-secret   Opaque                                1      4s
```

* Try creating a secret from the file you used to create a configmap. How can you modify your deployment to load its image colour setting from the secret instead of the configmap?
* What secret value is stored in the secret `user-token` in your personal namespace?

## Task 5: Cleanup

Ensure that your resources (deployments, secrets, configmaps, ...) are cleaned up, either by deleting them directly, or by deleting them using their YAML resource file.
