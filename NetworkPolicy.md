# Network Policy

- A network policy is a specification of how groups of pods are allowed to communicate with each other and other network endpoints.

- NetworkPolicy resources use labels to select pods and define rules which specify what traffic is allowed to the selected pods.

- Network policies are implemented by the network plugin

- By default, pods are non-isolated; they accept traffic from any source.

- Once there is any NetworkPolicy in a namespace selecting a particular pod, that pod will reject any connections that are not allowed by any NetworkPolicy. 

## The NetworkPolicy Resource

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

- podSelector - selects pods to which policy applies. 

- policyTypes - indicates whether the given policy applies to ingress, egress traffic from selected pods, or both.

- ingress - include a list of whitelist ingress rules.

    - Each rule allows traffic which matches both the from and ports sections. 

- egress -  list of whitelist egress rules.  

- isolates “role=db” pods in the “default” namespace for both ingress and egress traffic


- Ingress - allows connection on TCP port 6379 from:

    - any pod in the “default” namespace with the label “role=frontend”

    - any pod in a namespace with the label “project=myproject”

    - IP addresses in the ranges 172.17.0.0/16 except 172.17.1.0/24

- Egress - allow connection to 

    - any pod in the “default” namespace with the label “role=db” to CIDR 10.0.0.0/24 on TCP port 5978


## Behavior of to and from selectors

- selectors that can be specified in an ingress 'from' section or egress 'to' section:

## podSelector 

-  selects particular Pods in the same namespace as the NetworkPolicy.

## namespaceSelector 

-  selects particular namespaces for which all Pods should be allowed 

## namespaceSelector and podSelector:

```
  ...
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
      podSelector:
        matchLabels:
          role: client
  ...
```

- single from element allowing connections from Pods with the label role=client in namespaces with the label user=alice.

```
  ...
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
    - podSelector:
        matchLabels:
          role: client
  ...
```

- two elements in the from array, and allows connections from Pods in the local Namespace with the label role=client, or from any Pod in any namespace with the label user=alice.

## ipBlock 

- particular IP CIDR ranges to allow as ingress sources or egress destinations.

-  These should be cluster-external IPs, since Pod IPs are ephemeral and unpredictable.

Cluster ingress and egress mechanisms often require rewriting the source or destination IP of packets. In cases where this happens, it is not defined whether this happens before or after NetworkPolicy processing, and the behavior may be different for different combinations of network plugin, cloud provider, Service implementation

In the case of ingress, this means that in some cases you may be able to filter incoming packets based on the actual original source IP, while in other cases, the “source IP” that the NetworkPolicy acts on may be the IP of a LoadBalancer or of the Pod’s node, etc.

For egress, this means that connections from pods to Service IPs that get rewritten to cluster-external IPs may or may not be subject to ipBlock-based policies.


## Default policies
By default, if no policies exist in a namespace, then all ingress and egress traffic is allowed to and from pods in that namespace. The following examples let you change the default behavior in that namespace.


## Default deny all ingress traffic

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {} // selects all pods
  policyTypes:
  - Ingress
```

- policy does not change the default egress isolation behavior.

##  Default allow all ingress traffic

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all
spec:
  podSelector: {}
  ingress:
  - {}
  policyTypes:
  - Ingress
```

## Default deny all egress traffic

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Egress
```

##  Default allow all egress traffic

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all
spec:
  podSelector: {}
  egress:
  - {}
  policyTypes:
  - Egress
```

## Default deny all ingress and all egress traffic

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```