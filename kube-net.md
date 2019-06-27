## kubernetes

- Kubernetes is API-centric system. everything communicates with API.

## Pod

- Pod is group of containers scheduled together.

- Everything side the pod share network. Container see each other as localhost. They share the IP address. 

## Controller

- Controller watches kubernetes API and responds to changes. Replicaset, Service, DNS, kubelet.

- Controller makes reality match the intent.

## Labels

- Metadata attached to any API resource.

- String to string, key-value pair, used to identify and group objects. 
Ex: app name, tier(frontend/backend), stage (dev/test/prod)


## Annotations

- Data that rides along API object.

- Third party pieces that carry information about the object.

## Selector

- Selectors are counterparts of labels. Labels identifies the object. Selector select them based on labels. This similar to - select pods where app=mysql

- Provides loose coupling. User can manage groups how ever they need.

- Selectors are used in services, deployment, replicasets.

- app: store, role=fe

- app: store, stage=test

## IP-per-Pod

- Every VM is given an IP CIDR, every pod gets IP within that block.

- It is accessible from other pods, regardless of which VM they are on.

- Every pod has network namespace and virtual interfaces.

## Container to container communication

- Containers can reach each other's ports on localhost.


## Network namespace

## VM - eth0 

- eth0 is associated with root ns.

- root ns has its own context of network stack.

## pod1 netns

- Pod has its own network namespace.

- provides the illusion of its own localhost.

## Virtual eth device

- (root netns) vethxx - eth0 (pod1 netns) - pipe pair 

- Pods talk to root netns via pipe virtual eth device.


## Pod to Pod within same node

(pod1 netns)eth0 - vethxx - cbr0 - vethyy - eth0 (pod2 netns) 

- container bridge provides L2 connectivity between veths. (switch)

- Pod1 send message to Pod2

- IP packet src: pod1 IP dst: pod2 IP

- packet path: pod1 netns -> eth0 -> vethxx (root netns)
 -> cbr0 (arp) -> vethyy -> eth0 (pod2 netns)
 

## Pod to Pod in different VM/nodes (GCE)

- Kubernetes itself is abstracted how network is implemented, it could be L2, L3, Overlay.

(Pod1)(VM1) cbr0 - eth0 - (NETWORK) - (VM2)eth0 - cbr0 (Pod2)

- Pod1 sends message to Pod2

- IP packet src: pod1 IP dst: pod2 IP

- packet path:
    pod1 netns -> eth0 -> vethxx -> cbr0 (arp will fail, send to default route) - eth0 (VM1)

- --can-ip-forward is enabled on VM's 

- VM's act as routers, anti-spoofing is disabled so its does not drop the packet.

- set up static route for each vm

gcloud compute routes create vm2 --destination-range=x.y.z.0/24 --next-hop-instance=vm2

- (vm1) eth0 -> NETWORK -> (VM2)eth0 -> cb0 

- There is no encapsulation or overlay in GCE flow.


## Services

- Pod's IP changes as 

    - Scale up, Scale down 
    - Rolling updates
    - Pods crash or hang
    - VM's reboot

- Services provide abstration, gives mapping of virtual IP address to group of pods.

- Packets sent to VIP will be routed to one of pods IP associated with service.

```
apiVersion: v1
kind: Service
metadata:
    name: store-be
spec:
    selector: 
        app: store
        role: be
    ports:
    - name: http
      port: 80
```

- selector field chooses which pods to load balance.

- This creates a distributed load balancer.

- Allocates clusterIP

- Any traffic to this clusterIP is routed to one of the Pod.

- Endpoint object is created with same name as service gives mapping of service name to lists the pods IP addresses. This is continuously updated as pods ip address change.

## Pod to service

Pod sends message to Service

- IP packet src: pod1 dst: svc1

- pod1 eth0 -> vethxx -> cbr0 -> iptables

- iptables does a DNAT dst: pod99  

- conntrack remembers address translation - 5 tuple, this is reverse on return path.

- src: pod1 dst:pod99 -> eth0 

Return path src: pod99 dst: pod1

- eth0 -> iptables

- ip tables uses conntract to un-DNAT src: svc1

- src: svc1 dst: pod1 -> cbr0 -> vethxx -> (pod1) eth0

- kube-proxy maintains the iptables rules. 

- kube-proxy watches the API server for any changes in services and updates the iptable rules.


## DNS

- DNS is service inside cluster.

- When service is created DNS creates A and SRV records.

- DNS autoscale to cluster size. Its a ratio of nodes to cores.


## Pod to internet

- src: pod1 dst: 8.8.8.8

- cbr0 -> iptables 

- iptables does SNAT (masquerade) src: vm-interal ip

- eth0 - 1:1NAT 

- 1:1 NAT rewrites src: vm-external ip

return path

- 1:1NAT src: vm-interal ip

- iptables src: pod1 ip

- cbr0 -> vethxx

## Receiving external traffic

## L4 load balancer - service

```
kind: Service
spec:
    type: LoadBalancer
```

- cloud controller allocates load balancer sets up the routes.

- net lb - create forwarding rule to all VM's.

- VM port is opened for netlb to talk to VM's.

- cloud controller writes back to the service with loadbalancer ingress ip.

- This ip address can go to DNS (route53)

## External traffic to service

- src: client dst: LB

- netlb chooses a VM, there will be a imbalance as some VM's may have more Pods, some less. netlb is unware of pods, it just chooses a VM.

- imbalance problem is solved in iptables.

- packet comes to VM's IP table, it chooses the pod to which to send to. 

- if the pod is on another VM

- src: VM1 dst: pod2 (VM2)

return path

- src: pod2 dst: VM1

- VM1 does the reverse masquared

Problem is client address gets hidden, and there is double bounce in network.

## OnlyLocal

- iptables will always choose pod on the same VM.

- Risk imbalance, retain client ip.

- annotations: 
    service.beta.kubernetes.io/external-traffic: OnlyLocal

- if we have many pods than nodes, then imbalance is not great.

## Ingress - L7 http load balancer

- Ingress is built on top of services.

- set the type of service to NodePort.

- NodePort allocates port on every VM in cluster. That port forwards to service.

- create ingress object.

```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata: 
    name: test-ingress
spec:
    rules:
    - http:
        paths:
        - path: /customers
          backend:
            serviceName: customers-be
        - path: /products
          backend:
            serviceName: products-be
```

- http url can map to different service.

- cloud controller goes to cloud provider to allocate http load balancer, setup routes to all VM's, 

- writes back loadbalancer ingress ip address to ingress resource.


## External to ingress

- routes are setup between LB and all VM's

- On all VM's ports are opened for each NodePort service.

- src: client dst: LB path: /product

- LB chooses a VM. 

- src: LB dst: VM3

- http LB is forward proxy, it terminates the TCP session.

- client IP can be retrieved from http x-forwarding headers.

- Packet arrives on NodePort. 

- packet is sent to iptables.

- iptables will choose a pod.

- src: vm3 dst: pod3

return path

- src: pod3 dst: vm3

- vm3 iptables 

- src: vm3 dst: LB

- LB L src: LB dst: client

- there is double hop

## avoid double hop

- specify onlylocal annotation in the service.


- liveness probes
- graceful termination
- cloud health checks
- firewalls
- headless service
- ipam
- ssl

