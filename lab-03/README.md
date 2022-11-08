# Lab 3 - YAML

Although relatively easy in concept, this lab is a very important one when 
working with Kubernetes.  In this lab we introduce YAML as a way to describe 
objects in Kubernetes.  So far we only used specific `kubectl` commands to 
create a single pod.

If you are not very familiar with YAML check out the website below for a basic
introduction and its relevance for Kubernetes:
https://www.mirantis.com/blog/introduction-to-yaml-creating-a-kubernetes-deployment/

## Task 1: Creating a pod using YAML

In the previous labs we have used `kubectl run ...` to create a
pod object in Kubernetes, instead we could (and probably should) have also
used YAML.  So what would this look like?

The YAML code to replace `kubectl run ...` looks like
this:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

To create the object in Kubernetes simply copy the above content into a file,
for example `lab-03-pod.yml`.  After that simply issue the following
commmand:

```
kubectl apply -f lab-03-pod.yml

---

pod/nginx created
```

## Task 2: Deleting objects using YAML

Deleting an object is even easier, only requirement is that you have the orignal
YAML file at your disposal:

```
kubectl delete -f lab-03-pod.yml

---

pod "nginx" deleted
```

## Task 3: Multiple objects

When you have multiple objects you have different options:
* you can put multiple objects in the same YAML (using lists), we will cover 
this in a later lab
* you can have multiple YAML files inside a directory and `kubectl apply` the
directory (for example: `kubectl apply -f directory_full_of_yaml/`)

## Task 4: YAML and namespaces

Remember: you have all been assigned your own namespace to work in, and that your kubeconfig file contains an entry which will cause `kubectl` to target this namespace by default. Of course, it is possible to override this behaviour:
1. You can pass the `-n <NAMESPACE>` option to `kubectl`. Try to recreate the pod, but this time in the `default` namespace, like so: `kubectl apply -f lab-03-pod.yml -n default`. What happens?
2. You can specify the namespace in the metadata section of your resource YAML.

The second option warrants some further investigation. Create a copy of `lab-03-pod.yml` called `lab-03-pod-ns.yml` and include the namespace specification, like so (be sure to correctly replace the placeholder `<YOUR NAMESPACE>`):

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: <YOUR NAMESPACE>
spec:
  containers:
  - name: nginx
    image: nginx
```

Now apply the new file.

```
kubectl apply -f lab-03-pod-ns.yml

---

pod/nginx created
```

Of course, this will do the exact same thing as before; `kubectl` was already targeting your personal namespace, as dictated by your kubeconfig file.

* What happens when you try to deploy this modified YAML file into the default namespace? (`kubectl apply -f pod.yml -n default`)
* What happens when you try to change the namespace specification, e.g. to `default`?

## Task 5: Cleaning up

Ensure that your nginx pods are cleaned up, either by deleting them directly as before, or by deleting them using their YAML resource file.
