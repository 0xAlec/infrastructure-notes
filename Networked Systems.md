
## Service Mesh

*Data plane* - userspace proxies, intercepts calls between services 
*Userspace* - area of memory/storage, can be auth, session, browsing, etc.
*Control plane* - set of management processes, controls behavior of proxies and provides APIs for operators to manipulate and maintain the mesh

*Envoy* - high-performance, open-source proxy for microservice architecture. L7 proxy means it operates on OSI model's application layer and makes routing decision based on content of HTTP/gRPC requests. provides features like load balancing, service discovery, and traffic management.


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
L7 - based on content of request (session, content, type) - useful for when you want to route requests based on application logic
L4 - IP address and transport protocol (client IP, server IP, port #) - useful for when you want to route requests based on network topology or high-throughput low-latency workloads

*Target Groups* - listeners receive requests and decide which target group to forward the request. requests are then routed to instances, containers, or IP addresses. also performs health checks on targets

*API Gateway* - create, update, and publish APIs. compatible with REST architecture/HTTP. in contrast to ALBs which are typically used for gRPC requests. 

gRPC uses binary encoding, this is not natively supported by HTTP protocols. gRPC also allows for bi-directional streaming, flow control, and cancellation, which is not possible with REST.


