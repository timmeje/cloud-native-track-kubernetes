# Lab 9 - Blue / green deployments

## Task 1: Blue / green deployment

For this task we are creating 2 versions of 1 deployment. This means that we
can use the service to point to either of the deployments. This can be used for
example if you want to update your application. We are going to use the
container-info application again to demo this feature.

First we will create a blue deployment. This is the first and initial version
of our application.

Create a file `lab-9-deployment-blue.yml` with the following content.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: container-info-blue
  labels:
    app: container-info
spec:
  replicas: 3
  selector:
    matchLabels:
      app: container-info
  template:
    metadata:
      labels:
        app: container-info
        version: "blue"
    spec:
      containers:
      - name: container-info
        image: gluobe/container-info:blue
        ports:
        - containerPort: 80
```

As you can see we have added a new label called `version` with this label we can
select the correct deployment in the service we are creating in this task.

```
kubectl apply -f lab-9-deployment-blue.yml

---

deployment.apps/container-info-blue created
```

Now create the file `lab-9-service-blue-green.yml` with the following content.

```
kind: Service
apiVersion: v1
metadata:
  name: container-info
spec:
  selector:
    app: container-info
    version: "blue"
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

Notice that in the `spec:selector:version` we chose for `blue` this specifies
the version of the deployment we are going to expose.

```
kubectl apply -f lab-9-service-blue-green.yml

---

service/container-info created
```

After you exposed the `blue` version we are going to create a `green` version
of the application. This is done by creating a file
`lab-9-deployment-green.yml` with the following content.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: container-info-green
  labels:
    app: container-info
spec:
  replicas: 3
  selector:
    matchLabels:
      app: container-info
  template:
    metadata:
      labels:
        app: container-info
        version: "green"
    spec:
      containers:
      - name: container-info
        image: gluobe/container-info:green
        ports:
        - containerPort: 80
```

We changed some parameters from `blue` to `green`, notice that the image changed
as well. We are now deploying a `green` version.

```
kubectl apply -f lab-9-deployment-green.yml

---

deployment.apps/container-info-green created
```

If you list the deployments in your namespace you will see that we now have 2
deployments.

```
kubectl get deployment

---

NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
container-info-blue    3         3         3            3           12m
container-info-green   3         3         3            3           12m
```

A `blue` and a `green` version. Remember that our service is still pointing at
the `blue` version. In the next step we will update the service in a way that it
will point to the `green` version of our deployment.

Edit the `lab-09-service-blue-green.yml` file with the content below. You can 
also just clear the file and copy the following content.

```
kind: Service
apiVersion: v1
metadata:
  name: container-info
spec:
  selector:
    app: container-info
    version: "green"
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

The only thing that is updated in this file is basically the `version` this is
set to `green` now.

```
kubectl apply -f lab-09-service-blue-green.yml

---

service/container-info configured
```

Now reload the service in your browser and note how the colour changed.

We can now delete the blue deployment:

```
kubectl delete -f lab-09-deployment-blue.yml

---

deployment.apps "container-info-blue" deleted
```

You now only have one deployment running, the `green`:

```
kubectl get deployment

---

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
container-info-green   3/3     3            3           3m31s
```

Congratulations, you have successfully performed you first `blue / green` deployment!

## Task 2: Cleanup

Ensure that your resources are cleaned up, either by deleting them directly, or by deleting them using their YAML resource file.