# Policy and Security

- Microservices provide better agility, better scalability and better ability to reuse services.

- Istio provides solution for

    - to defend against man-in-middle attack, provide traffic encryption.

    - access control, mutual tls.

    - audit, who did what, at what time.

- Istio security mitigates both insider and external threats against your data, endpoints, communication and platform.

- The Istio security features provide strong identity, powerful policy, transparent TLS encryption, and authentication, authorization and audit (AAA) tools to protect your services and data. The goals of Istio security are:

    - Security by default: no changes needed for application code and infrastructure

    - Defense in depth: integrate with existing security systems to provide multiple layers of defense

    - Zero-trust network: build security solutions on untrusted networks

## Policies

- Rate limiting to dynamically limit the traffic to a service

- Denials, whitelists, and blacklists, to restrict access to services

- Header rewrites and redirects

Istio also lets you create your own policy adapters to add, for example, your own custom authorization behavior.


## High-level architecture

- Security in Istio involves multiple components:

    - Citadel for key and certificate management

    - Sidecar and perimeter proxies to implement secure communication between clients and servers

    - Pilot to distribute authentication policies and secure naming information to the proxies

    - Mixer to manage authorization and auditing

## Istio identity

- At the beginning of a service-to-service communication, the two parties must exchange credentials with their identity information for mutual authentication purposes. 

- On the client side, the server’s identity is checked against the secure naming information to see if it is an authorized runner of the service. On the server side, the server can determine what information the client can access based on the authorization policies, audit who accessed what at what time

-  Istio identity mode represents  human user, an individual service, or a group of services.

- Istio service identities on different platforms:

    - Kubernetes: Kubernetes service account

    - GKE/GCE: may use GCP service account

    - AWS: AWS IAM user/role account

    - On-premises (non-Kubernetes): user account, custom service account, The custom service account refers to the existing service account just like the identities that the customer’s Identity Directory manages.

## PKI

- The Istio PKI is built on top of Istio Citadel and securely provisions strong identities to every workload. 

- Istio uses X.509 certificates to carry the identities in SPIFFE format. 

- The PKI also automates the key & certificate rotation at scale.

## Kubernetes scenario

- Citadel watches the Kubernetes apiserver, creates a SPIFFE certificate and key pair for each of the existing and new service accounts. Citadel stores the certificate and key pairs as Kubernetes secrets.

- When you create a pod, Kubernetes mounts the certificate and key pair to the pod according to its service account via Kubernetes secret volume.

- Citadel watches the lifetime of each certificate, and automatically rotates the certificates by rewriting the Kubernetes secrets.

- Pilot generates the secure naming information, which defines what service account or accounts can run a certain service. Pilot then passes the secure naming information to the sidecar Envoy.

## On-premises machines scenario

- Citadel creates a gRPC service to take Certificate Signing Requests (CSRs).

- Node agent generates a private key and CSR, and sends the CSR with its credentials to Citadel for signing.

- Citadel validates the credentials carried with the CSR, and signs the CSR to generate the certificate.

- The node agent sends both the certificate received from Citadel and the private key to Envoy.

- The above CSR process repeats periodically for certificate and key rotation.

## Node agent in Kubernetes

- envoy - node agent - citadel

- using node agent in Kubernetes for certificate and key provisioning

- Citadel creates a gRPC service to take CSR requests.

- Envoy sends a certificate and key request via Envoy secret discovery service (SDS) API.

- Upon receiving the SDS request, the node agent creates the private key and CSR before sending the CSR with its credentials to Citadel for signing.

- Citadel validates the credentials carried in the CSR and signs the CSR to generate the certificate.

- The node agent sends the certificate received from Citadel and the private key to Envoy via the Envoy SDS API.

- The above CSR process repeats periodically for certificate and key rotation.

```
If Citadel is compromised, all its managed keys and certificates in the cluster may be exposed. We strongly recommend running Citadel in a dedicated namespace (for example, istio-citadel-ns), to restrict access to the cluster to only administrators.
```

## Example

- photo-frontend, photo-backend, and datastore. 

-  creates three namespaces: istio-citadel-ns, photo-ns, and datastore-ns.

- The photo SRE team creates two service accounts to run photo-frontend and photo-backend respectively in the photo-ns namespace. 

- The datastore SRE team creates one service account to run the datastore service in the datastore-ns namespace. 

- we need to enforce the service access control in Istio Mixer such that photo-frontend cannot access datastore.

## Authentication

-  two types of authentication:

- Transport authentication, also known as service-to-service authentication:

    - 