# Lab 10 - Liveness / readiness probes

## Task 1: Using readiness probe 

To test the readiness of a pod we are going to create the following file. Name 
the file `lab-10-probe-readiness.yml` and fill with the content below.

Pay special attention to the `readinessProbe` section.  This is the check that 
Kubernetes will perform continuously to see if a pod is ready or not.

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: probe-readiness
  name: probe-readiness
spec:
  containers:
  - name: probe-readiness
    image: busybox
    args:
    - /bin/sh
    - -c
    - sleep 6000
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/ready
      initialDelaySeconds: 5
      periodSeconds: 5
```

Apply the file in your namespace to create the `probe-readiness` pod.

```
kubectl apply -f lab-10-probe-readiness.yml

---

pod "probe-readiness" created
```

Now we need to check the state of the pod.

```
kubectl get pods

---

NAME              READY   STATUS    RESTARTS   AGE
probe-readiness   0/1     Running   0          10s
```

We see that the pod is running but not ready yet: `READY: 0/1`.

Before we are going to fix this readiness check we are going to create a 
service for this pod.

Create the file `lab-10-probes-service.yml` with this content.

```
kind: Service
apiVersion: v1
metadata:
  name: probes-service
spec:
  selector:
    app: probe-readiness
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

Apply the file and check out the service.

```
kubectl apply -f lab-10-probes-service.yml

---

service "probes-service" created
```

When we describe the service at this moment we won't see an endpoint, this is 
expected.  Kubernetes will only create an endpoint for pods when they are in the 
ready state.

```
kubectl describe service probes-service

---

Name:                     probes-service
Namespace:                ...
Labels:                   <none>
Annotations:              ...
Selector:                 app=probes
Type:                     NodePort
IP:                       10.105.221.153
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32748/TCP
Endpoints:
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

* Try to access your service as you would otherwise. What happens?
* Fix it! You need to ensure the readiness probe starts to succeed.
* Once you fixed it, check that there now is an available endpoint when describing the service again.

## Task 2: Using liveness probe 

Now we will simulate a situation where the pod becomes unhealthy (in other words 
the `livenessProbe` will fail).  If a pods becomes unhealthy Kubernetes will try 
to fix this by deleting the pod (and starting a new pod).

To test the liveness of a pod we are going to create the following file. Name 
the file `lab-10-probe-liveness.yml` and fill with the content below.

Pay special attention the the`livenessProbe` section.  This is the check that 
Kubernetes will perform continuously to see if a pod is alive/healthy or not.

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: probe-liveness
  name: probe-liveness
spec:
  containers:
  - name: probe-liveness
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 6000
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

Apply the file in your namespace to create the `probe-liveness` pod.

```
kubectl apply -f lab-10-probe-liveness.yml

---

pod/probe-liveness created
```

Now we need to check the state of the pod.

```
kubectl get pods

---

NAME             READY   STATUS    RESTARTS   AGE
probe-liveness   1/1     Running   0          10s
```

* What do you have to do to make the livenessprobe fail?
* What happens when you do that?

## Task 2: Cleanup

Ensure that your resources are cleaned up, either by deleting them directly, or by deleting them using their YAML resource file.