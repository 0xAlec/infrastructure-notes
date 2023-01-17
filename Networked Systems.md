
## Service Mesh

- *Data plane* - userspace proxies, intercepts calls between services 

- *Userspace* - area of memory/storage, can be auth, session, browsing, etc.

- *Control plane* - set of management processes, controls behavior of proxies and provides APIs for operators to manipulate and maintain the mesh

- *Envoy* - high-performance, open-source proxy for microservice architecture. L7 proxy means it operates on OSI model's application layer and makes routing decision based on content of HTTP/gRPC requests. provides features like load balancing, service discovery, and traffic management.

- *Target Groups* - listeners receive requests and decide which target group to forward the request. requests are then routed to instances, containers, or IP addresses. also performs health checks on targets. Used in load balancers. 

- *API Gateway* - create, update, and publish APIs. compatible with REST architecture/HTTP. in contrast to ALBs which are typically used for gRPC requests. 

The key difference a service mesh provides vs an API gateway or ingress proxy is that it manages requests and calls between microservices, compared to calls from the outside world into the cluster itself. 

Control plane provides TLS certificate issuance, metrics aggregation, etc. Control planes expose an API for users to modify and inspect the behavior of the data plane. An example of this is `kubectl` . 

gRPC uses binary encoding, this is not natively supported by HTTP protocols. gRPC also allows for bi-directional streaming, flow control, and cancellation, which is not possible with REST.


![linkerd](https://uploads-ssl.webflow.com/625ee9b2f6a4ec3997f9c11b/62a0d30e9625786b6d7c45d1_control-plane.png)
*An example of Linkerd's control + data plane. Each box is a Kubernetes pod, and the proxies run as a sidecar container in the pod.*

Depending on the mesh implementation, one proxy is added per node/host/VM. As a result, data plane proxies must be **fast** and **light** since we're adding 2 proxy hops to every call (one on server-side, one on client-side). These proxies will also consume resources (CPU/memory) in each pod, scaling linearly. Container orchestration is also necessary here to aggregate deployments and updates to the proxies.

Why add a service mesh? Well, it provides application-layer features like reliability (retries, timeouts, canaries), obervability (aggregation, metrics, latencies, request volumes, network topology), and security (mutual TLS, access management). 

Service meshes can provide a centralized and consistent way to manage these features across multiple instances, regardless of language, framework, or deployment platform. **Uniform across the stack and decoupled from application code**

It's also possible to combine service mesh with gRPC interceptors to provide features like authentication, authorization, metrics/tracing. While service meshes are typically implemented as a sidecar proxy for each service instance, interceptors are typically implemented as part of the service code and can add functionality to specific, individual service instances. 

**While service meshes can provide metrics like traffic + % of failed requests, internal metrics must be provided by application instrumentation specific to the individual service (e.g. interceptors).**


## OSI model

1. Physical Layer - raw bits of data
2. Data Link Layer - creates reliable link between two devices on a network
3. Network Layer - routing packets of data
4. Transport Layer - ensures data is delivered reliably and in order
5. Session Layer - stores session state
6. Presentation Layer - data compression, encryption, translation
7. Application Layer - provides services to user (email, file transfer, browsing)

*TCP* - Transmission Control Protocol, connection-oriented and establishes a reliable connection between devices before sending data. Ensures packets are sent in the correct order and lost packets are retransmitted. Use for applications that require high-level of reliability, such as file transfers and web browsing.

*UDP* - User Datagram Protocol, does not establish a connection and sends packets of data without checking if they are successfully received or not. Suitable for low-latency applications like video gaming or streaming.

## Types of Proxies + Traffic

- *Reverse proxy* - directs incoming requests to appropriate servers. reverse proxies can be used for load balancing, security, caching, SSL, URL rewriting, compression, and service discovery.
- *Forward proxy* - sits on a client's network and directs outgoing traffic to appropriate server. useful for enforcing security policies
- *Ingress proxy* - typically used in Kubernetes (and implemented as a pod), ingress proxies routes traffic to servers in k8s clusters based on the incoming request's URL or hostname. used to expose multiple services through a single IP or hostname, and provide a single entry point for external traffic into a cluster

North-south (*ingress/egress*) is terminology used to describe inbound requests (to a cluster), while east-west (*service to service*) is used to describe intracluster communication. Ingress proxies like KIC (*Kubernetes Ingress Controller*) is typically used to proxy request into and out of k8s clusters, while service meshes facilitate traffic between k8s services.

When should we use a reverse proxy vs an ingress proxy? Reverse proxies are general purpose while ingress proxies are specialized for Kubernetes clusters (and you can define routing rules inside the ingress resource). 

## Traffic Control

- Rate limiting - restricts # of requests by a given user in a time period
- Circuit breaker - prevent cascading failures by monitoring for service failures. e.g. when # of requests fail past a set threshold, return an immediate error response to clients as soon as the requests arrive, throttling traffic that would otherwise result in high latency and delayed timeouts which may cause an overall failure of the application

## Traffic Splitting

- *Debug routing* - deploy but allow access via session cookie, ID, etc. 
- *Canary deployment* - test stability of a new deployment, move a test group to the new version - if crashes or errors happen, return clients to the stable version. if successful, can start a gradual and controlled migration
- *A/B testing* - 50/50, choose based on KPIs
- *Blue green deployment* - downtime for upgrades is unacceptable. keep old version (blue) in production while deploy the new version (green) into the same production environment. use a canary deployment to incrementally move users. useful for zero downtime deployments + easy rollbacks.

## Load Balancing

Can provide load balancing at the application layer (load balancing requests to a load balancer) through reverse proxies like Envoy or ELB/ALB. 

Multiple load balancers can provide high availability, scalability, and geographic distribution (I believe AWS ELB/ALB is already configured for multi-region). However, this increases architecture complexity.

### **Application Load Balancer vs Network Load Balancer**

- ALB - L7, can route requests based on geographical location of the client. path-based routing, host-based routing, redirects, etc. **best suited for HTTP/HTTPS traffic**
- NLB - L4, Route53 latency-based routing to route traffic to optimal NLB endpoint based on IP protocol data. **best suited for routing TCP/UDP**

Load balancing at different network layers is optimized for different workloads:

- L7 - based on content of request (session, content, type) - useful for when you want to route requests based on application logic
- L4 - IP address and transport protocol (client IP, server IP, port #) - useful for when you want to route requests based on network topology or high-throughput low-latency workloads

