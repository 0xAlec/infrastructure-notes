
## Service Mesh

*Data plane* - userspace proxies, intercepts calls between services 

*Userspace* - area of memory/storage, can be auth, session, browsing, etc.

*Control plane* - set of management processes, controls behavior of proxies and provides APIs for operators to manipulate and maintain the mesh

*Envoy* - high-performance, open-source proxy for microservice architecture. L7 proxy means it operates on OSI model's application layer and makes routing decision based on content of HTTP/gRPC requests. provides features like load balancing, service discovery, and traffic management.

The key difference a service mesh provides vs an API gateway or ingress proxy is that it manages requests and calls between microservices, compared to calls from the outside world into the cluster itself. 

Control plane provides TLS certificate issuance, metrics aggregation, etc. Control planes expose an API for users to modify and inspect the behavior of the data plane. An example of this is `kubectl` . 

![[Pasted image 20230116164444.png]]
*An example of LinkedIn's control + data plane. Each box is a Kubernetes pod, and the proxies run as a sidecar container in the pod.*

Depending on the mesh implementation, one proxy is added per node/host/VM. As a result, data plane proxies must be **fast** and **light** since we're adding 2 proxy hops to every call (one on server-side, one on client-side). These proxies will also consume resources (CPU/memory) in each pod, scaling linearly. Container orchestration is also necessary here to aggregate deployments and updates to the proxies.

If this is the case, why add a mesh? Well, these meshes provide application-layer features like reliability (retries, timeouts, canaries), obervability (aggregation, metrics, latencies, request volumes, network topology), and security (mutual TLS, access management). 

Service meshes can provide a centralized and consistent way to manage these features across multiple instances, regardless of language, framework, or deployment platform. **Uniform across the stack and decoupled from application code**

It's also possible to combine service mesh with gRPC interceptors to provide features like authentication, authorization, metrics/tracing. Internal metrics must be provided by application instrumentation like interceptors.

While service meshes are typically implemented as a sidecar proxy for each service instance, interceptors are typically implemented as part of the service code and can add functionality to specific, individual service instances. 



## OSI model
1. Physical Layer - raw bits of data
2. Data Link Layer - creates reliable link between two devices on a network
3. Network Layer - routing packets of data
4. Transport Layer - ensures data is delivered reliably and in order
5. Session Layer - stores session state
6. Presentation Layer - data compression, encryption, translation
7. Application Layer - provides services to user (email, file transfer, browsing)

*TCP* - Transmission Control Protocol, connection-oriented and establishes a reliable connection between devices before sending data. Ensures packets are sent in the correct order and lost packets are retransmitted. Use for applications that require high-level of reliability, such as file transfers and web browsing.

*UDP* - User Datagram Protocol, does not establish a connection and sends pockets of data without checking if they are successfully received or not. Suitable for low-latency applications like video gaming or streaming.

*Reverse proxy* - directs incoming requests to appropriate servers. reverse proxies can be used for load balancing, security, caching, SSL, URL rewriting, compression, and service discovery.

Can provide load balancing at the application layer (load balancing requests to a load balancer) through reverse proxies like Envoy. 

Multiple load balancers can provide high availability, scalability, and geographic distribution (I believe AWS ELB/ALB is already configured for multi-region). However, this increases architecture complexity.

ALB - L7, can route requests based on geographical location of the client. path-based routing, host-based routing, redirects, etc. best suited for HTTP/HTTPS traffic
NLB - L4, Route53 latency-based routing to route traffic to optimal NLB endpoint based on IP protocol data. best suited for routing TCP/UDP

Load balancing at different network layers is optimized for different workloads:

- L7 - based on content of request (session, content, type) - useful for when you want to route requests based on application logic
- L4 - IP address and transport protocol (client IP, server IP, port #) - useful for when you want to route requests based on network topology or high-throughput low-latency workloads

*Target Groups* - listeners receive requests and decide which target group to forward the request. requests are then routed to instances, containers, or IP addresses. also performs health checks on targets

*API Gateway* - create, update, and publish APIs. compatible with REST architecture/HTTP. in contrast to ALBs which are typically used for gRPC requests. 

gRPC uses binary encoding, this is not natively supported by HTTP protocols. gRPC also allows for bi-directional streaming, flow control, and cancellation, which is not possible with REST.


