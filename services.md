# Services

- Every Pod has cluster-private-ip address.

- All Pods in network reach each other via this IP address without NAT.

- Containers within Pod reach each other via localhost.

- Since Pods are ephemeral, every time pod is recreated due to node going down, the pod gets a differnt IP address.

- Service provides an abstraction over set of Pods that provide same functionality providing them virtual IP that remains unchanged and a DNS name.

- Pods are configured to talk to service, service will automatically load balance the traffic to one of member pod.

```
apiVersion: apps/v1
kind: Deployment
metadata:
    name: mynginx
spec:
    selector:   
        matchLabels:
            run: mynginx
    replicas: 2
    template:
        metadata:
            labels:
                run: mynginx
        spec:
            containers:
            - name: mynginx
              image: nginx
              ports:
              - containerPort: 80
```

```
kubectl apply -f mynginx.yaml
kubectl get pods -l run=mynginx -o yaml | grep podIP  < see the pod ip of two replicas are different>
```

```
kubectl expose deployment mynginx
```

```
apiVersion: v1
kind: Service
metadata:
    name: mynginx
spec:
    ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
    selector:
        run: mynginx
```
- service are assigned a cluster-ip by default.

- selector determines the set of pods targeted by service.

- targetPort is the port on which container accepts the traffic.

- port is abstracted service port.

- default protocol is TCP


```
kubectl get svc mynginx
```

- Pods are exposed via Endpoints object

- The Endpoint controller will continuously scan for pods that match service selector, and will then post any updates to an Endpoint object also named mynginx  

## Service without selectors

- Useful for connecting to external service outside the cluster.

```
apiVersion: v1
kind: Service
metadata:
    name: mysvc
spec:
    ports:
    - port: 80
      targetPort: 9376
```

- Corresponding Endpoint object is not created.

- We can manually map service to network address and port.

```
apiVersion: v1
kind: Endpoints
metadata:
    name: mysvc
spec:
    - addresses:
        - ip: 192.0.2.42
      ports:
        - port: 9376
```

## kube-proxy

- Every node in cluster run kube-proxy. 

- kube-proxy is responsible for implementing virtual IP for services.

## iptables proxy mode

- kube-proxy watches the kubernetes control plane for any addition or removal of Services and Endpoint objects.

- For each service it installs iptable rules which captures the traffic to Service's cluster-ip and port and redirects to one of the pod.

- pod is chosen randomly.


## IPVS proxy mode

- kube-proxy watches Kubernetes Services and Endpoints, calls netlink interface to create IPVS rules accordingly and synchronizes IPVS rules with Kubernetes Services and Endpoints periodically. 

- The IPVS proxy mode is based on netfilter hook function uses hash table as the underlying data structure.

- IPVS provides more options for balancing traffic to backend Pods
Roud robin, least connection, destination hashing, source hashing, shortest expected delay


## Multi-port Services

```
apiVersion: v1
kind: Service
metadata:
    name: mysvc
spec:
    selector:
        run: myap
    ports:
    - name: http
      port: 80
      targetPort: 9376
    - name: https
      port: 443
      targetPort: 9377
```

- must give all of your ports, names.


## Choosing your own IP address

- specify your own cluster IP address with .spec.clusterIP

- configure service-cluster-ip-range CIDR range for the API server.

## Discovering services

- Environment variables

- DNS

## Environment varibles

- kubelet adds a set of environment variables for each active Service.

- {SVCNAME}_SERVICE_HOST

- {SVCNAME}_SERVICE_PORT

## DNS

- CoreDNS is install on cluster as DNS service.

- It watches the kubernetes API for new services and creates a set of DNS records for each one.

- Pods use DNS to resolve service by their DNS name to cluster-ip.

## Headless services

- .spec.clusterIP: None

- cluster IP is not allocated, kube-proxy does not load balance these services.

- With selectors - DNS return A records that point directly to pods backing the service.

- without selector - DNS system looks for or configures either

    - CNAME records for ExternalName-type services.

    - A records for any Endpoints that share a name with the service.


## service types

## clusterIP 

    - type: clusterIP

    - only reachable within cluster.

    - default service type.

## NodePort

    - type: NodePort

    - cluster IP is automatically created.

    - Exposes the service on every Node IP at a static port.

    - Accessed from outside the cluster by requesting <NodeIP>:<NodePort>

    - Node will forward the traffic to clusterip of the service.

    - NodePort is allocated from range --service-node-port-range flag (default: 30000-32767)

    - --nodeport-addresses flag in kube-proxy specify particular IP(s) to proxy the port

## LoadBalancer

    - type: LoadBalancer

    - Exposes service externally using cloud provider's load balancer.

    - NodePort and ClusterIP services are automatically created.

    - External load balancer will route traffic to NodePort and ClusterIP services.

```
apiVersion: v1
kind: Service
metadata:
    name: mysvc
spec: 
    selector:
        run: myapp
    ports:
    - port: 80
      targetPort: 9376
    type: LoadBalancer
```

    - The actual creation of the load balancer happens asynchronously, and information about the provisioned balancer will be published in the Service’s .status.loadBalancer field. 

## ExternalName

    -  ExternalName map a service to a DNS name

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

    - maps the my-service Service in the prod namespace to my.database.example.com

    - cluster DNS service will return a CNAME record.

    - redirection happens at the DNS level rather than via proxying 

## External IP

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 9376
  externalIPs:
  - 80.11.12.10
```

- my-service can be accessed by clients on "80.11.12.10:80" 


## Securing the Service

-  Before exposing the Service to the internet, you want to make sure the communication channel is secure. We need

    -  certificate for https

    - An nginx server configured to use the certificates

    - A secret that makes the certificates accessible to pods

```
#create a public private key pair
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /d/tmp/nginx.key -out /d/tmp/nginx.crt -subj "/CN=my-nginx/O=my-nginx"

#convert the keys to base64 encoding
cat /d/tmp/nginx.crt | base64
cat /d/tmp/nginx.key | base64
```   

- Use the output from the previous commands to create a yaml file as follows. The base64 encoded value should all be on a single line.

```
apiVersion: "v1"
kind: "Secret"
metadata:
  name: "nginxsecret"
  namespace: "default"
data:
  nginx.crt: "LS0tLS1"
  nginx.key: "LS0tLS"
```

```
kubectl apply -f nginxsecrets.yaml
```
```
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    protocol: TCP
    name: https
  selector:
    run: my-nginx
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 1
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      volumes:
      - name: secret-volume
        secret:
          secretName: nginxsecret
      containers:
      - name: nginxhttps
        image: nginx
        ports:
        - containerPort: 443
        - containerPort: 80
        volumeMounts:
        - mountPath: /etc/nginx/ssl
          name: secret-volume
```

- Each container has access to the keys through a volume mounted at /etc/nginx/ssl. This is setup before the nginx server is started.

