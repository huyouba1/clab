## CONTAINERlab + KinD
![](https://github.com/huyouba1/clab/blob/main/KinD.png)

## INFO


### 1. CONTAINERlab and KinD to deploy a new K8S cluster
The tool KinD can deploy a K8S cluster quicklly, but all the nodes usder the same subnet, so if want set the nodes under different subnets, CONTAINERlab can provide the network resources.

### 2. Deploy the K8S cluster with KinD:
```
$ git clone https://github.com/huyouba1/clab.git
$ kind create cluster --config=./kind-calico.yaml --image=harbor.int-guoyutec.com/kindest/node:v1.32.0
Creating cluster "clab-bgp-cplane-demo" ...
 ✓ Ensuring node image (kindest/node:v1.23.4) 
 ✓ Preparing nodes      
 ✓ Writing configuration 
 ✓ Starting control-plane ️ 
 ✓ Installing StorageClass 
 ✓ Joining worker nodes 
Set kubectl context to "kind-clab-bgp-cplane-demo"
You can now use your cluster with:
kubectl cluster-info --context kind-clab-bgp-cplane-demo
Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 
``` 
Once done ,we can get a K8S cluster without CNI.
```
$ kubectl get no   -o wide
NAME                                 STATUS     ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION       CONTAINER-RUNTIME
clab-bgp-cplane-demo-control-plane   NotReady   control-plane   44m   v1.32.0   172.18.0.3    <none>        Debian GNU/Linux 12 (bookworm)   5.15.0-126-generic   containerd://1.7.24
clab-bgp-cplane-demo-worker          NotReady   <none>          44m   v1.32.0   172.18.0.4    <none>        Debian GNU/Linux 12 (bookworm)   5.15.0-126-generic   containerd://1.7.24
clab-bgp-cplane-demo-worker2         NotReady   <none>          44m   v1.32.0   172.18.0.2    <none>        Debian GNU/Linux 12 (bookworm)   5.15.0-126-generic   containerd://1.7.24
clab-bgp-cplane-demo-worker3         NotReady   <none>          44m   v1.32.0   172.18.0.5    <none>        Debian GNU/Linux 12 (bookworm)   5.15.0-126-generic   containerd://1.7.24
```

![](https://github.com/huyouba1/clab/blob/main/CONTAINERlab.png)
### 3. Deploy the network resources with CONTAINERlab
```
$  brctl  show
bridge name     bridge id               STP enabled     interfaces
br-2b7ad9611ba2         8000.0242b189bdc8       no              veth15866b3
                                                        veth78c8dbb
                                                        veth8b82849
                                                        vethe235be7
docker0         8000.0242ed193481       no

$ brctl addbr br-leaf0
$ ifconfig br-leaf0 up
$ brctl addbr br-leaf1
$ ifconfig br-leaf1 up
$ brctl show
bridge name     bridge id               STP enabled     interfaces
br-2b7ad9611ba2         8000.0242b189bdc8       no              veth15866b3
                                                        veth78c8dbb
                                                        veth8b82849
                                                        vethe235be7
br-leaf0                8000.0e2e85d8ec9b       no
br-leaf1                8000.1220d82f6be4       no
docker0         8000.0242ed193481       no

$ clab  deploy -t topo.yaml
INFO[0000] Containerlab v0.60.1 started
INFO[0000] Parsing & checking topology file: topo.yaml
INFO[0000] Creating docker network: Name="clab", IPv4Subnet="172.20.20.0/24", IPv6Subnet="3fff:172:20:20::/64", MTU=1500
INFO[0000] Creating lab directory: /root/clab-master/clab-bgp-cplane-demo
WARN[0000] node clab-bgp-cplane-demo-control-plane referenced in namespace sharing not found in topology definition, considering it an external dependency.
WARN[0000] node clab-bgp-cplane-demo-worker referenced in namespace sharing not found in topology definition, considering it an external dependency.
WARN[0000] node clab-bgp-cplane-demo-worker2 referenced in namespace sharing not found in topology definition, considering it an external dependency.
WARN[0000] node clab-bgp-cplane-demo-worker3 referenced in namespace sharing not found in topology definition, considering it an external dependency.
INFO[0000] Creating container: "spine1"
INFO[0000] Creating container: "leaf1"
INFO[0000] Creating container: "spine0"
INFO[0006] Creating container: "leaf0"
INFO[0008] Creating container: "server4"
INFO[0008] Created link: leaf1:eth2 <--> spine1:eth2
INFO[0008] Created link: leaf1:eth3 <--> br-leaf1:br-leaf1-net2
INFO[0008] Creating container: "server2"
INFO[0009] Created link: leaf1:eth1 <--> spine0:eth2
INFO[0009] Creating container: "server1"
INFO[0009] Created link: br-leaf1:br-leaf1-net1 <--> server4:net0
INFO[0009] Created link: br-leaf0:br-leaf0-net1 <--> server2:net0
INFO[0010] Created link: leaf0:eth1 <--> spine0:eth1
INFO[0010] Created link: leaf0:eth2 <--> spine1:eth1
INFO[0010] Created link: leaf0:eth3 <--> br-leaf0:br-leaf0-net2
INFO[0010] Creating container: "server3"
INFO[0010] Created link: br-leaf0:br-leaf0-net0 <--> server1:net0
INFO[0010] Created link: br-leaf1:br-leaf1-net0 <--> server3:net0
INFO[0010] Executed command "ip addr add 172.18.0.4/16 dev net0" on the node "server2". stdout:
INFO[0010] Executed command "ip route replace default via 172.18.0.1" on the node "server2". stdout:
INFO[0010] Executed command "ip addr add 172.18.0.2/16 dev net0" on the node "server1". stdout:
INFO[0010] Executed command "ip route replace default via 172.18.0.1" on the node "server1". stdout:
INFO[0010] Executed command "ip addr add 172.18.0.5/16 dev net0" on the node "server3". stdout:
INFO[0010] Executed command "ip route replace default via 172.18.0.1" on the node "server3". stdout:
INFO[0010] Executed command "ip addr add 172.18.0.3/16 dev net0" on the node "server4". stdout:
INFO[0010] Executed command "ip route replace default via 172.18.0.1" on the node "server4". stdout:
INFO[0010] Adding containerlab host entries to /etc/hosts file
INFO[0010] Adding ssh config for containerlab nodes
╭──────────────────────────────┬────────────────────────────┬─────────┬───────────────────╮
│             Name             │         Kind/Image         │  State  │   IPv4/6 Address  │
├──────────────────────────────┼────────────────────────────┼─────────┼───────────────────┤
│ clab-bgp-cplane-demo-leaf0   │ linux                      │ running │ 172.20.20.5       │
│                              │ vyos/vyos:1.2.8            │         │ 3fff:172:20:20::5 │
├──────────────────────────────┼────────────────────────────┼─────────┼───────────────────┤
│ clab-bgp-cplane-demo-leaf1   │ linux                      │ running │ 172.20.20.3       │
│                              │ vyos/vyos:1.2.8            │         │ 3fff:172:20:20::3 │
├──────────────────────────────┼────────────────────────────┼─────────┼───────────────────┤
│ clab-bgp-cplane-demo-server1 │ linux                      │ running │ N/A               │
│                              │ burlyluo/nettoolbox:latest │         │ N/A               │
├──────────────────────────────┼────────────────────────────┼─────────┼───────────────────┤
│ clab-bgp-cplane-demo-server2 │ linux                      │ running │ N/A               │
│                              │ burlyluo/nettoolbox:latest │         │ N/A               │
├──────────────────────────────┼────────────────────────────┼─────────┼───────────────────┤
│ clab-bgp-cplane-demo-server3 │ linux                      │ running │ N/A               │
│                              │ burlyluo/nettoolbox:latest │         │ N/A               │
├──────────────────────────────┼────────────────────────────┼─────────┼───────────────────┤
│ clab-bgp-cplane-demo-server4 │ linux                      │ running │ N/A               │
│                              │ burlyluo/nettoolbox:latest │         │ N/A               │
├──────────────────────────────┼────────────────────────────┼─────────┼───────────────────┤
│ clab-bgp-cplane-demo-spine0  │ linux                      │ running │ 172.20.20.4       │
│                              │ vyos/vyos:1.2.8            │         │ 3fff:172:20:20::4 │
├──────────────────────────────┼────────────────────────────┼─────────┼───────────────────┤
│ clab-bgp-cplane-demo-spine1  │ linux                      │ running │ 172.20.20.2       │
│                              │ vyos/vyos:1.2.8            │         │ 3fff:172:20:20::2 │
╰──────────────────────────────┴────────────────────────────┴─────────┴───────────────────╯
```

### 4. There is key note that how to combine the network resources which create by CONTAINERlab with KinD:
    server1:
      kind: linux
      image: burlyluo/nettoolbox:latest
      network-mode: container:control-plane    # bind contaier name
      exec:
      - ip addr add 192.168.5.10/24 dev net0     # 1. use the container mode network, the address is container subnet.
      - ip route replace default via 192.168.5.1 # 2. replace the default gateway due to the most of CNI will select the default interface nic as the CNI interface. container network gateway
 
 the router's configuration can be found at [startup-config](https://github.com/huyouba1/clab/tree/main/startup-config)
So with this logical, we can get a full topo as below:

![](https://github.com/huyouba1/clab/blob/main/TOPO.png)



### 5. Apply the CNI
```
# kubectl apply -f calico.yaml 
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
poddisruptionbudget.policy/calico-kube-controllers created
# 
```
