# Networking

- container to container within Pod

- Pod to Pod

- Pod to Service

- External to internal

## Container to container

- Containers can reach each other's ports on localhost.

## Pod to pod

-  IP-per-pod, pods can communicate without NAT.

- ip-forwarding-enabled on VMs so that each VM has an extra 256 IP addresses that get routed to it. This is in addition to the 'main' IP address assigned to the VM that is NAT-ed for Internet access. 

- GCE itself does not know anything about Pod IPs, though. This means that when a pod tries to egress beyond GCE's project the packets must be SNAT'ed (masqueraded) to the VM's IP, which GCE recognizes and allows.

## Pod to service

- Service has virtual IP which clients can access backend pods.

- Each node runs a kube-proxy process which programs iptables rules to trap access to service IPs and redirect them to the correct backends.

## External to internal

- Set up external load balancers which target all nodes in a cluster.

- When traffic arrives at a node it is recognized as being part of a particular Service and routed to an appropriate backend Pod. 

- This does mean that some traffic will get double-bounced on the network. 