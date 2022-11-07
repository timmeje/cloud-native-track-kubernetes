# Lab 05 - Services

A Kubernetes Service is an abstraction which defines a logical set of Pods and a
policy by which to access them - sometimes called a micro-service. The set of
Pods targeted by a Service is (usually) determined by a Label Selector.

As an example, consider an image-processing backend which is running with 3
replicas. Those replicas are fungible - frontends do not care which backend they
use. While the actual Pods that compose the backend set may change, the frontend
clients should not need to be aware of that or keep track of the list of
backends themselves. The Service abstraction enables this decoupling.

## Task 1: Creating your first service

Create a file `lab-05-deployment.yml` with the YAML for the deployment using the
content below:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: container-info
  labels:
    app: container-info
spec:
  replicas: 2
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
        image: gluobe/container-info:blue
        ports:
        - containerPort: 80
```

Create the deployment using the above file:

```
kubectl apply -f lab-05-deployment.yml

---

deployment "container-info" created
```

Verfiy that this is working:

```
kubectl get pods

---

NAME                              READY     STATUS    RESTARTS   AGE
container-info-5998b79944-gh57z   1/1       Running   0          21s
container-info-5998b79944-t4h6n   1/1       Running   0          21s
```

Now create a file `lab-05-service.yml` with the YAML for the service using the
content below:

```
kind: Service
apiVersion: v1
metadata:
  name: container-info
spec:
  selector:
    app: container-info
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

Create the service using the above file:

```
kubectl apply -f lab-05-service.yml

---

service "container-info" created
```

Check that the service has been created succesfully:

```
kubectl get service

---

NAME             TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
container-info   NodePort   10.152.183.231   <none>        80:32207/TCP   6m54s
```

## Task 2: Accessing your first service

In previous labs we always used the `port-forward` method to expose a pod, as 
this method only allows us to connect to a single pod we will use a different 
method when wofking with services backed by multiple, replicated pods.

* How can you access your service? Try to figure out what host and port you have to connect to! Hints:
  * You can `describe` most (if not all) Kubernetes objects using `kubectl` to get detailed info on them
  * A `NodePort` service will expose a port on all nodes in a Kubernetes cluster, but our cluster only has one node! It has a publicly accessible hostname/IP address.

Once you've figured out how to connect to the `NodePort` service using your browser, you will see how Kubernetes will forward your requests randomly to either of your backend pods by refreshing.

> NOTE: when you refresh you should see a different "avatar" for each pod, the 
> avatar is generated based on the container name, however browsers tend to 
> cache pretty hard, so you might need to clear your cache/cookies before 
> hitting refresh to see a different avatar.

If you cannot see different avatars, use can use the command below to test with 
`curl` instead.  The command will `curl` the application 10 times and will list 
the container name.  You should see that it hits different pods:

```
for i in {1..10}; do curl -s http://<HOSTNAME or IP>:<NODEPORT> | egrep "<td>container-info"; done

          <td>container-info-6d9747978f-2x5r7</td>
          <td>container-info-6d9747978f-2x5r7</td>
          <td>container-info-6d9747978f-c9twz</td>
          <td>container-info-6d9747978f-c9twz</td>
          <td>container-info-6d9747978f-sqvtw</td>
          <td>container-info-6d9747978f-sqvtw</td>
          <td>container-info-6d9747978f-c9twz</td>
          <td>container-info-6d9747978f-sqvtw</td>
          <td>container-info-6d9747978f-2x5r7</td>
          <td>container-info-6d9747978f-sqvtw</td>
```

## Task 3: Cleaning up

Ensure that your resources (deployments, services, ...) are cleaned up, either by deleting them directly, or by deleting them using their YAML resource file.