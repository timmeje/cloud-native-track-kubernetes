# Lab 1 - Nodes

A node is a worker machine in Kubernetes, previously known as a minion. A node
may be a VM or physical machine, depending on the cluster.  A node is where your 
application containers will run.

Each node contains the services necessary to run pods and is managed by the 
master components. The services on a node include the container runtime, kubelet 
and kube-proxy.

## Task 1: Listing nodes

To see which nodes are part of your Kubernetes cluster run the
`kubectl get nodes` command:

```
kubectl get nodes

---

NAME                     STATUS   ROLES    AGE   VERSION
kbck8s.timmeje.gluo.io   Ready    <none>   47m   v1.25.3
```

As we are using minikube we only have a single node.  Below is the output of a
3 node cluster:

```
kubectl get nodes

---

NAME                                        STATUS    ROLES     AGE       VERSION
gke-kbc-steven-default-pool-b82ee1c9-5n9j   Ready     <none>    20m       v1.11.7-gke.4
gke-kbc-steven-default-pool-b82ee1c9-6wnx   Ready     <none>    20m       v1.11.7-gke.4
gke-kbc-steven-default-pool-b82ee1c9-x13z   Ready     <none>    20m       v1.11.7-gke.4
```

We can use the `kubectl get nodes -o wide` command to get some additional
information about the nodes:

```
kubectl get nodes -o wide

---

NAME                     STATUS   ROLES    AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
kbck8s.timmeje.gluo.io   Ready    <none>   47m   v1.25.3   172.31.12.132   <none>        Ubuntu 22.04.1 LTS   5.15.0-1022-aws   containerd://1.6.6
```

## Task 2: Getting detailed node information

If we want more detailed information about a node we have to use the
`kubectl describe nodes <node_name>` command, for example:

```
kubectl describe nodes kbck8s.timmeje.gluo.io

---

Name:               kbck8s.timmeje.gluo.io
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=kbck8s.timmeje.gluo.io
                    kubernetes.io/os=linux
                    microk8s.io/cluster=true
                    node.kubernetes.io/microk8s-controlplane=microk8s-controlplane
Annotations:        node.alpha.kubernetes.io/ttl: 0
                    projectcalico.org/IPv4Address: 172.31.12.132/20
                    projectcalico.org/IPv4VXLANTunnelAddr: 10.1.70.0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Mon, 07 Nov 2022 12:29:28 +0100
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  kbck8s.timmeje.gluo.io
  AcquireTime:     <unset>
  RenewTime:       Mon, 07 Nov 2022 13:18:48 +0100
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Mon, 07 Nov 2022 12:30:18 +0100   Mon, 07 Nov 2022 12:30:18 +0100   CalicoIsUp                   Calico is running on this node
  MemoryPressure       False   Mon, 07 Nov 2022 13:17:35 +0100   Mon, 07 Nov 2022 12:29:28 +0100   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Mon, 07 Nov 2022 13:17:35 +0100   Mon, 07 Nov 2022 12:29:28 +0100   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Mon, 07 Nov 2022 13:17:35 +0100   Mon, 07 Nov 2022 12:29:28 +0100   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Mon, 07 Nov 2022 13:17:35 +0100   Mon, 07 Nov 2022 12:31:18 +0100   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  172.31.12.132
  Hostname:    kbck8s.timmeje.gluo.io
Capacity:
  cpu:                4
  ephemeral-storage:  64847800Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             16195060Ki
  pods:               110
Allocatable:
  cpu:                4
  ephemeral-storage:  63799224Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             16092660Ki
  pods:               110
System Info:
  Machine ID:                 ec236f3e833c25d9db1776c61af9a83a
  System UUID:                ec236f3e-833c-25d9-db17-76c61af9a83a
  Boot ID:                    39aba443-541b-4f6c-abf1-a9565b2c92cb
  Kernel Version:             5.15.0-1022-aws
  OS Image:                   Ubuntu 22.04.1 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.6.6
  Kubelet Version:            v1.25.3
  Kube-Proxy Version:         v1.25.3
Non-terminated Pods:          (3 in total)
  Namespace                   Name                                        CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                        ------------  ----------  ---------------  -------------  ---
  kube-system                 calico-node-885h4                           250m (6%)     0 (0%)      0 (0%)           0 (0%)         49m
  kube-system                 coredns-d489fb88-drd5n                      100m (2%)     0 (0%)      70Mi (0%)        170Mi (1%)     48m
  kube-system                 calico-kube-controllers-597f868657-r6wvw    0 (0%)        0 (0%)      0 (0%)           0 (0%)         49m
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests   Limits
  --------           --------   ------
  cpu                350m (8%)  0 (0%)
  memory             70Mi (0%)  170Mi (1%)
  ephemeral-storage  0 (0%)     0 (0%)
  hugepages-1Gi      0 (0%)     0 (0%)
  hugepages-2Mi      0 (0%)     0 (0%)
Events:              <none>
```

Read through the output above to see all the information that is available about
the node.

The `kubectl describe nodes <node_name>` is a very useful tool when
troubleshooting a node that is failing.  Going through the output (especially 
the event section) will, in almost all cases, give a  good indication why the 
node is not behaving as it should.