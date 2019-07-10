
- Istio generates detailed telemetry for all service communications within a mesh.

- This telemetry provides observability of service behavior, empowering operators to troubleshoot, maintain, and optimize their applications – without imposing any additional burdens on service developers.

- Metrics -  Istio generates a set of service metrics based on the four “golden signals” of monitoring (latency, traffic, errors, and saturation). 

- Distributed Traces - Istio generates distributed trace spans for each service, providing operators with a detailed understanding of call flows and service dependencies within a mesh.

- Access Logs  - As traffic flows into a service within a mesh, Istio can generate a full record of each request, including source and destination metadata. This information enables operators to audit service behavior down to the individual workload instance level.

## Metrics

- To monitor service behavior, Istio generates metrics for all service traffic in, out, and within an Istio service mesh. 

- They give info on overall volume of traffic, the error rates within the traffic, and the response times for requests.

- Istio components export metrics on their own internal behaviors to provide insight on the health and function of the mesh control plane.

- Operators select how and when to collect metrics, as well as how detailed the metrics themselves should be. 

## Proxy-level Metrics

- Each proxy generates a rich set of metrics about all traffic passing through the proxy (both inbound and outbound). The proxies also provide detailed statistics about the administrative functions of the proxy itself, including configuration and health information.

- 
