# flannel

- Flannel is responsible for providing a layer 3 IPv4 network between multiple nodes in a cluster. 

- Flannel creates overlay network over host network.

- Kubernetes assigns a unique IP address to each pod.

- This works well on Google Compute where each host is assigned a /24 for use by individual pods. 

- Things are not as easy on other cloud providers where a host cannot get an entire subnet to itself. 

- flannel aims to solve this problem by creating an overlay mesh network that provisions a subnet to each server.

- An overlay network is first configured with an IP range and the size of the subnet for each host. 

- For example, one could configure the overlay to use 10.100.0.0/16 and each host to receive a /24 subnet. 

-  Host A could then receive 10.100.5.0/24 and host B could get 10.100.18.0/24.

- Flannel uses either the Kubernetes API or etcd directly to store the network configuration and the allocated subnets.

- To perform the encapsulation, flannel utilizes Universal TAP/TUN devices (in TUN mode) and proxies the IP fragments between the TUN device and a UDP socket.


- Packets are forwarded using one of several backend mechanisms including VXLAN and various cloud integrations.

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

## Backends

- Flannel may be paired with several different backends. Once set, the backend should not be changed at runtime.

- VXLAN is the recommended choice. 

- host-gw is recommended for more experienced users who want the performance improvement and whose infrastructure support it (typically it can't be used in cloud environments). 

- UDP is suggested for debugging only or for very old kernels that don't support VXLAN.


## VXLAN

- Use in-kernel VXLAN to encapsulate the packets.

    - Type (string): vxlan

    - VNI (number): VXLAN Identifier (VNI) to be used. On Linux, defaults to 1. 

    - Port (number): UDP port to use for sending encapsulated packets. On Linux, defaults to kernel default, currently 8472.

    - GBP (Boolean): Enable VXLAN Group Based Policy. Defaults to false. 

    - DirectRouting (Boolean): Enable direct routes (like host-gw) when the hosts are on the same subnet. VXLAN will only be used to encapsulate packets to hosts on different subnets. Defaults to false.

https://www.youtube.com/watch?v=3eAVHt3lyuM

https://blog.laputa.io/kubernetes-flannel-networking-6a1cb1f8ec7c


