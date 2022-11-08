# Lab 7 - Persistent Volumes

Persistent volumes can be used to store stateful/persistent data, this means
that when a pods is replaced its successor will still have the data available.
This can be used when you want to run persistent databases in pods and/or when
you want to share data between different pods.

For this lab we will make a persistent volume that we fill with images, those
images will then be served by our application (pods).

## Task 0: Setting up the persistent volume

Before starting this lab, we'll demonstrate how to set up the persistent volume as a group (your personal kubeconfig does not contain credentials which have the necessary permissions).

## Task 1: Claim the persistent volume

Now we need to 'claim' the persistent volume. This way we can bind
the persistent volume to a deployment. First we need to create a file
`lab-07-pvc.yml` with the following content.

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: lab-07-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadOnlyMany
  resources:
    requests:
      storage: 1Gi
```

And apply it with the following command.

```
kubectl apply -f lab-07-pvc.yml

---

persistentvolumeclaim/lab-07-claim created
```

You can confirm that it is created with the following command.

```
kubectl get pvc

---

NAME           STATUS   VOLUME          CAPACITY   ACCESS MODES   STORAGECLASS   AGE
lab-07-claim   Bound    lab-09-volume   1Gi        ROX            manual         28s
```

## Task 4: Mounting a persistent volume in pods

Now we are able to use the PersistentVolumeClaim in a pod. The following
steps will show you how you can do this.

First we need to create a file that describes our pod. This will be
`lab-07-deployment.yml` with the following content.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: meme-persistent
  labels:
    app: meme-persistent
spec:
  replicas: 2
  selector:
    matchLabels:
      app: meme-persistent
  template:
    metadata:
      labels:
        app: meme-persistent
    spec:
      volumes:
      - name: lab-07-volume
        persistentVolumeClaim:
          claimName: lab-07-claim
      containers:
      - name: meme-persistent
        image: gluobe/meme-persistent:v1
        ports:
        - containerPort: 80
        volumeMounts:
          - mountPath: "/var/www/html/images"
            name: lab-07-volume
```

Apply the file with the kubectl command:

```
kubectl create -f lab-07-deployment.yml

---

deployment.apps/meme-persistent created
```

Be sure that the pods are running.

```
kubectl get pods

---

meme-persistent-84cdc97446-dc2v4   1/1     Running   0          31s
meme-persistent-84cdc97446-dh59c   1/1     Running   0          31s
meme-persistent-84cdc97446-tq8h8   1/1     Running   0          31s
```

Create a service to access the application, create a file `lab-07-service.yml`
with the content below:

```
kind: Service
apiVersion: v1
metadata:
  name: meme-persistent
spec:
  selector:
    app: meme-persistent
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

Apply the file:

```
kubectl apply -f lab-07-service.yml

---

service/meme-persistent created
```

Check the application by finding the node port and connecting in your browser.

When you refresh the page you should see that again the avatar changes (there
are three pods), but the images remain the same (the PHP script will simply
show all the images from the persistent volume directory).

## Task 5: Cleanup

Ensure that your resources (deployments, services, persistentvolumeclaims, ...) are cleaned up, either by deleting them directly, or by deleting them using their YAML resource file.

> NOTE: we have now deleted our namespace, but as our persistent volume is not 
> namespaced and still exists.  So even if you deleted your application your data 
> would still be available.