# istio

- Developers are using microservices to architect their application.

- Istio provides a uniform way to connect, secure and observe services.

- Istio provides service discovery, traffic routing, and load balancing for your service mesh without having to update your services.

- Istio simplifies configuration of service-level properties like timeouts and retries, and makes it straightforward to set up tasks like staged rollouts with percentage-based traffic splits.

- Istio uses sidecar proxies to capture traffic and, where possible, automatically program the networking layer to route traffic through those proxies without any changes to the deployed application code. 

- In Kubernetes, the proxies are injected into pods and traffic is captured by programming iptables rules. 

- Once the sidecar proxies are injected and traffic routing is programmed, Istio can mediate all traffic. 


## Service mesh

- Service mesh is network of services that make the application and interaction between them.

- Service mesh needs to

    - Service Discovery

    - Load Balancing

    - Failure recovery

    - metrics

    - monitoring

Other requirements

    - A/B testing

    - canary rollouts

    - rate limiting

    - access control

    - authentication, authorization, and encryption

## Istio

- Secure service-to-service communication in a cluster with strong identity-based authentication and authorization and encryption.

- Istio’s robust tracing, monitoring, and logging features give you deep insights into your service mesh deployment. 

- Istio’s Mixer component is responsible for policy controls and telemetry collection. It provides backend abstraction and intermediation, insulating the rest of Istio from the implementation details of individual infrastructure backends, and giving operators fine-grained control over all interactions between the mesh and infrastructure backends.


## Platform support

- Kubernetes

- Nomad with Consul

- running on individual virtual machines

## Architecture

- Logically split into data plane and a control plane.

## Data plane

- composed of a set of intelligent proxies (Envoy) deployed as sidecars.

- These proxies mediate and control all network communication between microservices along with Mixer, a general-purpose policy and telemetry hub.

- The control plane manages and configures the proxies to route traffic. Additionally, the control plane configures Mixers to enforce policies and collect telemetry.

Service A - Envoy

Mixer

Pilot Galley Citadel

## Envoy

- mediate all inbound and outbound traffic for all services in the service mesh. 

- Built-in features

    - Dynamic service discovery

    - Load balancing

    - TLS termination

    - HTTP/2 and gRPC proxies

    - Circuit breakers

    - Health checks

    - Staged rollouts with %-based traffic split

    - Fault injection

    - Rich metrics

- Envoy is deployed as a sidecar to the relevant service in the same Kubernetes pod.

- This deployment allows Istio to extract a wealth of signals about traffic behavior as attributes. Istio can, in turn, use these attributes in Mixer to enforce policy decisions, and send them to monitoring systems to provide information about the behavior of the entire mesh.

- Istio capabilities to an existing deployment with no need to rearchitect or rewrite code. 


## Mixer

- Mixer enforces access control and usage policies across the service mesh, and collects telemetry data from the Envoy proxy and other services. 

- The proxy extracts request level attributes, and sends them to Mixer for evaluation. 

- Mixer includes a flexible plugin model. This model enables Istio to interface with a variety of host environments and infrastructure backends.

## Pilot

- Pilot provides service discovery for the Envoy sidecars, traffic management capabilities for intelligent routing (e.g., A/B tests, canary rollouts, etc.), and resiliency (timeouts, retries, circuit breakers, etc.).

- Pilot converts high level routing rules that control traffic behavior into Envoy-specific configurations, and propagates them to the sidecars at runtime.

- Pilot abstracts platform-specific service discovery mechanisms and synthesizes them into a standard format that any sidecar conforming with the Envoy data plane APIs can consume. 

- This loose coupling allows Istio to run on multiple environments such as Kubernetes, Consul, or Nomad

## Citadel

- Citadel enables strong service-to-service and end-user authentication with built-in identity and credential management.

- enforce policies based on service identity rather than on relatively unstable layer 3 or layer 4 network identifiers. 

## Galley

- Galley is Istio’s configuration validation, ingestion, processing and distribution component. 

- It is responsible for insulating the rest of the Istio components from the details of obtaining user configuration from the underlying platform (e.g. Kubernetes).


```
the greatest need is the ability to extend the policy system, to integrate with other sources of policy and control, and to propagate signals about mesh behavior to other systems for analysis. 
```


# Traffic Management

- Istio’s traffic management model relies on the following two components:

    - Pilot , the core traffic management component.

    - Envoy proxies, which enforce configurations and policies set through Pilot.

- These components enable

    - Service discovery

    - Load balancing

    - Traffic routing and control

## Pilot: Core traffic management

- Pilot 

- Abstract API

- Plateform API - kubernetes, mesos, cloudfoundry...

- Rules API

- Network API

- Envoy API - Envoy Proxy.. 

- Pilot maintains an abstract model of all the services in the mesh. Platform-specific adapters in Pilot translate the abstract model appropriately for your platform. 

- For example, the Kubernetes adapter implements controllers to watch the Kubernetes API server for changes to pod registration information and service resources. 

- The Kubernetes adapter translates this data for the abstract model.

- Pilot uses the abstract model to generate appropriate Envoy-specific configurations to let Envoy proxies know about one another in the mesh through the Envoy API.

- use Istio’s Traffic Management API to instruct Pilot to refine the Envoy configuration to exercise more granular control over the traffic in your service mesh.

## Envoy proxies

- Envoy proxies are the only Istio components that interact with data plane traffic. 

- Envoy proxies route the data plane traffic across the mesh and enforce the configurations and traffic rules without the services having to be aware of them. 

- Envoy proxies are deployed as sidecars to services,


### Service discovery and load balancing

- Istio service discovery leverages the service discovery features provided by platforms like Kubernetes for container-based applications. 

- The platform starts a new instance of a service which notifies its platform adapter.

- The platform adapter registers the instance with the Pilot abstract model.

- Pilot distributes traffic rules and configurations to the Envoy proxies to account for the change.

- Using the abstract model, Pilot configures the Envoy proxies to perform load balancing for service requests, replacing any underlying platform-specific load balancing feature. 

Istio supports the following load balancing methods:

- Round robin: Requests are forwarded to instances in the pool in turn, and the algorithm instructs the load balancer to go back to the top of the pool and repeat.

- Random: Requests are forwarded at random to instances in the pool.

- Weighted: Requests are forwarded to instances in the pool according to a specific percentage.

- Least requests: Requests are forwarded to instances with the least number of requests.

### Traffic routing and configuration

- Virtual services - Use a virtual service to configure an ordered list of routing rules to control how Envoy proxies route requests for a service within an Istio service mesh.

- Destination rules - Use destination rules to configure the policies you want Istio to apply to a request after enforcing the routing rules in your virtual service.

- Gateways - Use gateways to configure how the Envoy proxies load balance HTTP, TCP, or gRPC traffic.

- Service entries - Use a service entry to add an entry to Istio’s abstract model that configures external dependencies of the mesh.

- Sidecars - Use a sidecar to configure the scope of the Envoy proxies to enable certain features, like namespace isolation.

configure fine-grained traffic control for a range of use cases:

- Configure ingress traffic, enforce traffic policing, perform a traffic rewrite.

- Set up load balancers and define service subsets as destinations in the mesh.

- Set up canary rollouts, circuit breakers, timeouts, and retries to test network resilience.

- Configure TLS settings and outlier detection.

### Traffic routing use cases

### Routing traffic to multiple versions of a service

- Typically, requests sent to services use a service’s hostname or IP address, and clients sending requests don’t distinguish between different versions of the service.

- With Istio, because the Envoy proxy intercepts and forwards all requests and responses between the clients and the services, you can use routing rules with service subsets in a virtual service to configure the routing rules for multiple versions of a service.

- Service subsets are used to label all instances that correspond to a specific version of a service. Before you configure routing rules, the Envoy proxies use round-robin load balancing across all service instances, regardless of their subset. After you configure routing rules for traffic to reach specific subsets, the Envoy proxies route traffic to the subset according to the rule but again use round-robin to route traffic across the instances of each subset.

- A/B testing

- virtual service to specify a routing rule that sends 25% of requests to instances in the v2 subset, and sends the remaining 75% of requests to instances in the v1 subset.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 75
    - destination:
        host: reviews
        subset: v2
      weight: 25
```

### Canary rollouts with autoscaling

- Container orchestration platforms like Docker or Kubernetes support canary rollouts, but they use instance scaling to manage traffic distribution, which quickly becomes complex, especially in a production environment that requires autoscaling.

- With Istio, you can configure traffic routing and instance deployment as independent functions. The number of instances implementing the services can scale up and down based on traffic load without referring to version traffic routing at all. This makes managing a canary version that includes autoscaling a much simpler problem.

### Virtual services

- A virtual service is a resource you can use to configure how Envoy proxies route requests to a service within an Istio service mesh.

- Configure each application service version as a subset and add a corresponding destination rule to determine the set of pods or VMs belonging to these subsets.
Configure traffic rules in combination with gateways to control ingress and egress traffic

- Add multiple match conditions to a virtual service configuration to eliminate redundant rules.

- Configure traffic routes to your application services using DNS names. These DNS names support wildcard prefixes or CIDR prefixes to create a single rule for all matching services.

- Address one or more application services through a single virtual service. If your mesh uses Kubernetes, for example, you can configure a virtual service to handle all services in a specific namespace.

### Route requests to a subset

- configures the my-vtl-svc virtual service to route requests to the v1 subset of the my-svc service:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-vtl-svc
spec:
  hosts:
    - "*.my-co.org"
    http:
      - route:
        - destination:
            host: my-svc
            subset: v1
```

- virtual service handles routing for any DNS name ending with .my-co.org.

- under route, which specifies the routing rule’s configuration, and destination, which specifies the routing rule’s destination, host: my-svc specifies the destination’s host. If you are running on Kubernetes, then my-svc is the name of a Kubernetes service.

- The destination’s host must exist in the service registry. To use external services as destinations, use service entries to add those services to the registry.

### Route requests to services in a Kubernetes namespace

- If you use short names, the destinations must be in the same namespace as the virtual service. If you use fully qualified domain names, the destinations can be in any namespace.

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-namespace
spec:
  hosts:
    - my-namespace.com
  http:
  - match:
    - uri:
        prefix: /svc-1
    route:
    - destination:
        host: svc-1.my-namespace.svc.cluster.local
  - match:
    - uri:
        prefix: /svc-2
    route:
    - destination:
        host: svc-2.my-namespace.svc.cluster.local
```

## Routing rules

- A virtual service consists of an ordered list of routing rules to define the paths that requests follow within the mesh. 

- A routing rule consists of a destination and zero or more conditions

-  You can also use routing rules to perform some actions on the traffic, for example:

    - Append or remove headers.

    - Rewrite the URL.

    - Set a retry policy.

## Routing rule for HTTP traffic

- shows a virtual service that specifies two HTTP traffic routing rules. The first rule includes a match condition with a regular expression to check if the username “jason” is in the request’s cookie. If the request matches this condition, the rule sends traffic to the v2 subset of the my-svc service. Otherwise, the second rule sends traffic to the v1 subset of the my-svc service.

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-vtl-svc
spec:
  hosts:
    - "*"
 http:
 - match:
   - headers:
       cookie:
         regex: "^(.*?;)?(user=jason)(;.*)?$"
   route:
     - destination:
         host: my-svc
         subset: v2
 - route:
   - destination:
       host: my-svc
       subset: v1
```

## Match a condition

- you can restrict traffic to specific client workloads by using labels.

- The following rule only applies to requests coming from instances of the reviews service:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
      sourceLabels:
        app: reviews
    route:
    ...
```

## Conditions based on HTTP headers

- rule only applies to an incoming request that includes a custom end-user header containing the exact jason string:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    ...
```

## Match request URI

- only applies to a request if the URI path starts with /api/v1:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: productpage
spec:
  hosts:
    - productpage
  http:
  - match:
    - uri:
        prefix: /api/v1
    route:
    ...
```

## Multiple match conditions

- nesting of the conditions in the routing rule to specify whether AND or OR semantics apply.

- To specify AND semantics, you nest multiple conditions in a single section of match.

- For example, the following rule applies only to requests that come from an instance of the reviews service in the v2 subset AND only if the requests include the custom end-user header that contains the exact jason string:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
    - sourceLabels:
        app: reviews
        version: v2
      headers:
        end-user:
          exact: jason
    route:
    ...
```

- To specify OR conditions, you place multiple conditions in separate sections of match.

- For example, the following rule applies to requests from instances of the reviews service in the v2 subset, OR to requests with the custom end-user header containing the jason exact string:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
    - sourceLabels:
        app: reviews
        version: v2
    - headers:
        end-user:
          exact: jason
    route:
    ...
```
- The dash indicates two separate matches as opposed to one match with multiple conditions.

## Routing rule precedence

- Multiple rules for a given destination in a configuration file are evaluated in the order they appear. The first rule on the list has the highest priority.

## Precedence example with 2 rules

- The first rule sends all requests for the reviews service that include the Foo header with the bar value to the v2 subset. The second rule sends all remaining requests to the v1 subset:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        Foo:
          exact: bar
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1

```

- In this example, the header-based rule has the higher priority because it comes first in the configuration file.

## Destination rules

-  destination rules configure the set of policies that Envoy proxies apply to a request at a specific destination.

- Destination rules are applied after the routing rules are evaluated.

## Load balancing 3 subsets

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: my-destination-rule
spec:
  host: my-svc
  trafficPolicy:
    loadBalancer:
      simple: RANDOM
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
  - name: v3
    labels:
      version: v3

```
- As shown above, you can specify multiple policies in a single destination rule. In this example, the default policy is defined above the subsets field. The v2 specific policy is defined in the corresponding subset’s field.

## Service subsets

- Service subsets subdivide and label the instances of a service. 

- To define the divisions and labels, use the subsets section in destination rules. 

- For example, you can use subsets to configure the following traffic routing scenarios:

  - Use subsets to route traffic to different versions of a service.

  - Use subsets to route traffic to the same service in different environments.

- You use service subsets in the routing rules of virtual services to control the traffic to your services.

## Gateways

- You use a gateway to manage inbound and outbound traffic for your mesh. 

- Gateway configurations apply to Envoy proxies that are running at the edge of the mesh, which means that the Envoy proxies are not running as service sidecars. 

- You can use a gateway to configure workload labels for your existing network tasks, including:

  - Firewall functions

  - Caching

  - Authentication

  - Network address translation

  - IP address management

- Gateways are primarily used to manage ingress traffic, but you can also use a gateway to configure an egress gateway. 

- All traffic enters the mesh through an ingress gateway workload. To configure the traffic, use an Istio gateway and a virtual service.

- You bind the virtual service to the gateway to use standard Istio routing rules to control HTTP requests and TCP traffic entering the mesh.

## Configure a gateway for external HTTPS traffic

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: ext-host-gwy
spec:
  selector:
    app: my-gateway-controller
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    hosts:
    - ext-host
    tls:
      mode: SIMPLE
      serverCertificate: /tmp/tls.crt
      privateKey: /tmp/tls.key
```

- This gateway configuration lets HTTPS traffic from ext-host into the mesh on port 443, but doesn’t specify any routing for the traffic.

## Bind a gateway to a virtual service

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: virtual-svc
spec:
  hosts:
    - ext-svc
  gateways:
    - ext-host-gwy
```

## Service entries

- A service entry is used to add an entry to Istio’s abstract model, or service registry, that Istio maintains internally. After you add the service entry, the Envoy proxies can send traffic to the service as if it was a service in your mesh. 

  - Redirect and forward traffic for external destinations, such as APIs consumed from the web, or traffic to services in legacy infrastructure.

  - Define retry, timeout, and fault injection policies for external destinations.

  - Add a service running in a Virtual Machine (VM) to the mesh to expand your mesh.

  - Logically add services from a different cluster to the mesh to configure a multicluster Istio mesh on Kubernetes.

  - Access secure external services over plain text ports, to configure Envoy to perform TLS Origination .

  - Ensure, together with an egress gateway, that all external services are accessed through a single exit point.   

## Add an external dependency securely

- mesh-external service entry adds the ext-resource external dependency to Istio’s service registry:

```
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: svc-entry
spec:
  hosts:
  - ext-resource.com
  ports:
  - number: 443
    name: https
    protocol: HTTPS
  location: MESH_EXTERNAL
  resolution: DNS
```

- You must specify the external resource using the hosts key. You can qualify it fully or use a wildcard domain name. 

- Configuring a service entry can be enough to call an external service, but typically you configure either, or both, a virtual service or destination rule to control traffic in a more granular way. You can configure traffic for a service entry in the same way you configure traffic for a service in the mesh

## Secure the connection with mutual TLS

- secure the connection to the ext-resource external service

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: ext-res-dr
spec:
  host: ext-resource.com
  trafficPolicy:
    tls:
      mode: MUTUAL
      clientCertificate: /etc/certs/myclientcert.pem
      privateKey: /etc/certs/client_private_key.pem
      caCertificates: /etc/certs/rootcacerts.pem
```

- Together, the svc-entry service entry and the ext-res-dr destination rule configure a connection for traffic to the ext-resource external dependency using port 443 and mutual TLS.

## Sidecars

You can use a sidecar configuration to do the following:

  - Fine-tune the set of ports and protocols that an Envoy proxy accepts.

  - Limit the set of services that the Envoy proxy can reach.

## Enable namespace isolation

- the following Sidecar configures all services in the bookinfo namespace to only reach services running in the same namespace thanks to the ./* value of the hosts: field:

```
apiVersion: networking.istio.io/v1alpha3
kind: Sidecar
metadata:
  name: default
  namespace: bookinfo
spec:
  egress:
  - hosts:
    - "./*"
```

## Network resilience and testing

- Istio provides opt-in failure recovery features that you can configure dynamically at runtime through the Istio traffic management rules. With these features, the service mesh can tolerate failing nodes and Istio can prevent localized failures from cascading to other nodes:

- Timeouts and retries

  - A timeout is the amount of time that Istio waits for a response to a request. A retry is an attempt to complete an operation multiple times if it fails. You can set defaults and specify request-level overrides for both timeouts and retries or for one or the other.

- Circuit breakers

  - Circuit breakers prevent your application from stalling as it waits for an upstream service to respond. You can configure a circuit breaker based on a number of conditions, such as connection and request limits.

- Fault injection

  - Fault injection is a testing method that introduces errors into a system to ensure that it can withstand and recover from error conditions. You can inject faults at the application layer, rather than the network layer, to get more relevant results.

- Fault tolerance

  - You can use Istio failure recovery features to complement application-level fault tolerance libraries in situations where their behaviors don’t conflict.

## Timeouts and retries

- The default timeout for HTTP requests is 15 seconds. You can configure a virtual service with a routing rule to override the default:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
    - ratings
  http:
  - route:
    - destination:
        host: ratings
        subset: v1
    timeout: 10s
```

## Set number and timeouts for retries

- You can specify the maximum number of retries for an HTTP request in a virtual service, and you can provide specific timeouts for the retries to ensure that the calling service gets a response, either success or failure, within a predictable time frame.

- Envoy proxies automatically add variable jitter between your retries to minimize the potential impact of retries on an overloaded upstream service.

- The following virtual service configures three attempts with a 2-second timeout:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
    - ratings
  http:
  - route:
    - destination:
        host: ratings
        subset: v1
    retries:
      attempts: 3
      perTryTimeout: 2s

```

## Circuit breakers

- While retries let your application recover from transient errors, a circuit breaker pattern prevents your application from stalling as it waits for an upstream service to respond. By configuring a circuit breaker pattern, you allow your application to fail fast and handle the error appropriately, for example, by triggering an alert. You can configure a simple circuit breaker pattern based on a number of conditions such as connection and request limits.

## Limit connections to 100

The following destination rule sets a limit of 100 connections for the reviews service workloads of the v1 subset:

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
    trafficPolicy:
      connectionPool:
        tcp:
          maxConnections: 100
```

## Fault injection

- With Istio, you can use application-layer fault injection instead of killing pods, delaying packets, or corrupting packets at the TCP layer. You can inject more relevant failures at the application layer, such as HTTP error codes, to test the resilience of an application.

- You can inject two types of faults:

  - Delays: Delays are timing failures. They mimic increased network latency or an overloaded upstream service.

  - Aborts: Aborts are crash failures. They mimic failures in upstream services. Aborts usually manifest in the form of HTTP error codes or TCP connection failures.

## Introduce a 5 second delay in 10% of requests

- configure a virtual service to introduce a 5 second delay for 10% of the requests to the ratings service.

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - fault:
      delay:
        percentage:
          value: 0.1
        fixedDelay: 5s
    route:
    - destination:
        host: ratings
        subset: v1

```

## Return an HTTP 400 error code for 10% of requests

-  configure an abort instead to terminate a request and simulate a failure.

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - fault:
      abort:
        percentage:
          value: 0.1
        httpStatus: 400
    route:
    - destination:
        host: ratings
        subset: v1
```

## Combine delay and abort faults

-  configuration introduces a delay of 5 seconds for all requests from the v2 subset of the ratings service to the v1 subset of the ratings service and an abort for 10% of them:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
    - sourceLabels:
        app: reviews
        version: v2
    fault:
      delay:
        fixedDelay: 5s
      abort:
        percentage:
          value: 0.1
        httpStatus: 400
    route:
    - destination:
        host: ratings
        subset: v1
```

## Compatibility with application-level fault handling

- Istio failure recovery features are completely transparent to the application. Applications don’t know if an Envoy sidecar proxy is handling failures for a called upstream service, before returning a response.

- When you use application-level fault tolerance libraries and Envoy proxy failure recovery policies at the same time, you need to keep in mind that both work independently, and therefore might conflict.















