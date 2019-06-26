# Ingress

- Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. 

- Traffic routing is controlled by rules defined on the ingress controller.

- Ingress provides load balancing, SSL termination and name-based virtual hosting.

- Ingress controller is responsible for fulfulling the ingress, usually with load balancer.

- Ingress resource contains rules for directing HTTP traffic.

```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata: 
    name: test-ingress
    annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
spec:
    rules:
    - http:
        paths:
        - path: /testpath
          backend:
            serviceName: test
            servicePort: 80
```

## Ingress rules

- host: if no host is specified, rules apply to all incoming request. If host is provided, the rules apply to that host.

- list of paths: each of which are associated with backend defined by serviceName and servicePort.

## Default Backend

- An Ingress with no rules sends all traffic to a single default backend. 

- The default backend is typically a configuration option of the Ingress controller

- If none of the hosts or paths match the HTTP request in the Ingress objects, the traffic is routed to your default backend.

## Types of Ingress

## Single service ingress

- exposing single service

- specifying default backend with no rules.

```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata: 
    name: test-ingress
spec: 
    backend:
        serviceName: testsvc
        servicePort: 80
```


- kubectl apply -f ingress.yaml

- kubectl get ingress test-ingress

- shows IP allocated by the Ingress controller to satisfy this Ingress.

## Simple fan out

- Routes traffic from a single IP address to more than one service, based on HTTP URI.

- Ingress allows you to keep the number of load balancer to minimum.

```
http:
    paths:
    - path: /foo
    backend:
        serviceName: service1
        servicePort: 4200
    - path: /bar
    backend:
        serviceName: service2
        servicePort: 8080
```
- The Ingress controller provisions an load balancer that satisfies the Ingress, as long as the services (s1, s2) exist. 

## Name based virtual hosting

- Name-based virtual hosts support routing HTTP traffic to multiple host names at the same IP address.

```
foo.bar.com --|                 |-> foo.bar.com s1:80
              | 178.91.123.132  |
bar.foo.com --|                 |-> bar.foo.com s2:80
```

```
  rules:
  - host: foo.bar.com
    http:
      paths:
      - backend:
          serviceName: service1
          servicePort: 80
  - host: bar.foo.com
    http:
      paths:
      - backend:
          serviceName: service2
          servicePort: 80
```

- If you create an Ingress resource without any hosts defined in the rules, then any web traffic to the IP address of your Ingress controller can be matched without a name based virtual host being required.

## TLS

- 


