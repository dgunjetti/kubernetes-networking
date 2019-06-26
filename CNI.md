# CNI

## How are containers made

- containers are made of 

Namespaces
 - Process
   Network
   Mount
   UTS
   IPC 
   User

CGroups
- CPU 
  Memory

Layered Filesystem
- AUSF


## Linux Network Namespace

- Create container network isolation instances.

- Own networking stack, routing table, Virtual interfaces, L2 isolation.

- Tool used to work with network ns - iproute2 

- Network ns are stored in /var/run/netns

- there are two types of network namespaces

   - root namespace (ip link)

   - non-root namespace (ip netns .. ip link)

- ip netns add namespace1

- ip netns

- ls -l /var/run/netns

## Kubernetes Networking

        Linux Host 
Pod1 Namespace     Pod2 Namespace
Container1          Container3
Container2          Container4
eth0 10.0.0.5       eth0 10.0.0.6

vethxx              vethxx
docker0 - 10.0.0.1

eth0 <-> internet

## container networking interface (CNI)

- CNI concerns itself only creating network connectivity for containers when containers are created, and removing network connectivity when containers are deleted.

- It consists of a specification and libraries for writing plugins to configure network interfaces in Linux containers, along with a number of supported plugins.

- CNI acts as glue between orchestrator (runtime) and network implementation (plugin)

- CNI Plugin implement CNI Spec.

- Orchestrator calls CNI plugin to program the network namespace.

- CNI plugins have standard interface, so orchestrator is abstracted from which plugin is actually used underneth.

- Orchestrator and Plugin use CNI Library.

## CNI

https://github.com/containernetworking

Git repo has two parts.

- Plugin

- CNI
    - Spec
    - API/Library

## Spec
specification defines the basic flow and configuration format of network operations.
how plugin is given data by runtime and how status is reported back to runtime.


## configuration format

- JSON based configuration.

- It can contain standard key/value pair and plugin specific key/value pair.

- Configuration is fed to plugin on stdin on each operation.

- Stored on disk or by the runtime.

- configuration is fed to each plugin operation.

{
  "cniVersion":
  "name:
  "type": ptp
  "ipam": {
    "type": "host-local",
    "subnet": "10.0.0.0/24"
  }
}

## execution flow

- Basic commands: ADD DEL and VERSION

- ADD - add container to given network

- DEL - remove container from given network

- VERSION - return version of plugin.


## Plugins

- Plugins are executables

- Spawned(fork/exec) by runtime when network operation is desired, when container is created/deleted.

- Operations are fed by environment variables and configuration via stdin

- Plugins are also fed container-specific data via stdin - name of container, namespace, bandwidth requirements, QoS, portmapping - map container internal port to that of host

- Plugins report structured (JSON) result via stdout back to runtime - IP address, interfaces created, routes and DNS info

- Some CNI network plugins, maintained by the containernetworking team.

main - interface creating, L2 connectivity on host.
- bridge, ipvlan, macvlan, ptp, host-device, windows

ipam - ip address allocation
- dhcp, host-local, static

meta - other plugins,  bandwidth or qos guarantees
- flannel, tuning, portmap, bandwidth

## 3rd Party plugins

- Project calico

- Flannel

- Canal


## Kubernetes with Project Calico network plugin

create a container in kubernetes.

- API server calls Kubelet to create pod.

- Kubelet will create network namespace for pod - pause container or infrastructure container.

- Kubelet will call CNI with ADD command using CNI API library. kubelet - calico CNI

- Calico CNI will query API Server for labels and annotations.

- Calico CNI calls Calico IPAM to get IP address.

- Calico IPAM queries centralized datastore for IP address, returns IP address to Calico CNI.

- Calico CNI proceeds to network the Pod.

- Create veth pair, eth0 on pod side, calixxx (some hash) on host side.

- Add ip address to eth0, add default route.

- Configure sysctl on host side.

- Add route to container on host.

- Calico CNI reports to kubelet that container is networked.

- Kubelet reports pod IP address to API server.