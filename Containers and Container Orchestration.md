
Adoption of containers and container orchestration technologies allow proliferation of modern microservice-based architectures.

### What problem does Docker solve?

Containers solve the *packaging problem* - non-network runtime dependencies bundled into a standardized container allow developers to ensure programs can run on any machine, regardless of dependencies, frameworks, languages, etc. Increases the reproducibility of workloads.

### What problem does Kubernetes solve?

Container orchestration solves the problem of mapping containers to machines that execute these containers. Using container orchestration technologies allow you to minimize the deploy-time and infrastructure costs of 1 -> 10 -> 100 services. 

## Containers

Isolated namespaces and restricted processes
- **Namespaces provide the first and most straightforward form of isolation**: processes running within a container cannot see, and even less affect, processes running in another container, or in the host system.
- Control Groups are another key component of Linux Containers. They implement resource accounting and limiting. They provide many useful metrics, but they also help ensure that each container gets its fair share of memory, CPU, disk I/O; and, more importantly, that a single container cannot bring the system down by exhausting one of those resources.
- Each container has its own network stack. You can enable networking through exposing public ports for your containers. 
[Container runtimes](https://iximiuz.com/en/posts/implementing-container-runtime-shim/?z=100) - dockerd, containerd, runc
Just some processes running on a Linux host - not necessarily an operating system

Docker breakdown:
- dockerd - higher-level daemon in front of containerd daemon
- docker - cli client to interact with dockerd
It can:
- build/push/pull/scan images
- launch/pause/inspect/kill containers
- create networks/port forward

![Layered Docker architecture: docker (cli) -> dockerd -> containerd -> containerd-shim -> runc](https://iximiuz.com/journey-from-containerization-to-orchestration-and-beyond/docker-containerd-runc-2000-opt.png)
*Docker architecture*

## Container orchestration

*Pods*
- Allows packaging of sidecars with main containers
- Native communication between containers in a pod (e.g. via `localhost`)
- Containers can remain isolated

*Deployments*, a Kubernetes abstraction, allow you to specify the desired number of Pod replicas. Kubernetes is declarative by nature; users specify the desired end state and Kubernetes figures out the implementation details. In the example of scaling, the desired end state is defined in a manifest/resource, and pod distribution happens automatically across cluster nodes.

![[Pasted image 20230116194332.png]]
*Kubernetes deployments*

*ReplicaSet*
- A Kubernetes **resource**
- Ensures a specified number of pod replicas are running at all times
- Define a desired state for a *set* of pods, rather than an individual one.
- If a pod dies or becomes unavailable, the ReplicaSet will create a new pod to replace it
- If the number of running pods exceeds the desired number, the ReplicaSet will delete the extra pods.
- Self-healing, monitors health of pods and restarts/replaces unresponsive ones

### Service Discovery Problem

Problem: clients need to figure out the IP address and port it should use. In modern web services, there are multiple copies of a service in production at the same time.

Solution: service discovery. common solutions include a load balancer/reverse proxy like NGINX or HAProxy, using a single endpoint for the client which can route subsequent traffic to multiple instances of services. reverse proxies and load balancers utilize a *service registry* to determine the location and availability of a service. (a load balancer's routing rules are dynamically updated depending on this service registry)

**Load balancers = single point of failure, potential throughput bottleneck + extra network hop on request path**

![[Pasted image 20230116195312.png]]
*Server-side Service discovery*

Client-side and [round robin DNS](https://en.wikipedia.org/wiki/Round-robin_DNS) can also be used as an alternative (not studying this further as it doesn't seem to be industry standard)

Kubernetes has built-in service discovery! 

*Kubernetes service*
- A **microservice**
- Defines a set of pods and policies to access them
- pods are ephemeral
- name must include a valid DNS label name

Kubernetes service discovery is *similar* to DNS service discovery, but introduces the concept of `clusterIP` , which can be used to access pods in the service. Clients are able to learn the `clusterIP` for a service by inspecting its environment variables. When a pod is created, it is automatically assigned a unique IP address within the cluster. Services are also assigned unique IP addresses, and they can be used to access the pods that they represent.

```
Upon a pod startup, for every running service Kubernetes injects a couple of env variables looking like `<service-name>_SERVICE_HOST` and `<service-name>_SERVICE_PORT`.
```

[Example](https://kubernetes.io/docs/concepts/services-networking/service/#environment-variables):

```
REDIS_PRIMARY_SERVICE_HOST=10.0.0.11
REDIS_PRIMARY_SERVICE_PORT=6379
REDIS_PRIMARY_PORT=tcp://10.0.0.11:6379
REDIS_PRIMARY_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_PRIMARY_PORT_6379_TCP_PROTO=tcp
REDIS_PRIMARY_PORT_6379_TCP_PORT=6379
REDIS_PRIMARY_PORT_6379_TCP_ADDR=10.0.0.11
```

![[Pasted image 20230116200624.png]]
*This results in an architecture very similar to a load balancer/reverse proxy in front of the service.*

Managed offerings like Amazon EKS (Elastic Kubernetes Service) allow automatic provisioning, scaling, and maintaining the control plane and worker nodes to meet application demand.

In Amazon EKS, the service discovery is implemented using the Kubernetes DNS, which is automatically deployed and configured when the cluster is created. The DNS service runs as a pod inside the cluster and provides the service discovery functionality. Additionally, the Amazon EKS also uses AWS's internal DNS service (Route 53) to provide a seamless service discovery across the different availability zones.

### Blue/green and canary deployments in Kubernetes


![[Pasted image 20230116194213.png]]
*Blue/green and canary deployments in Kubernetes*

*Argo rollouts*
- Open-source Kubernetes extension for advanced deployments
- Manage the creation, scaling, and deletion of `ReplicaSet` objects.
- Useful for:
	- Canary deployments
	- Blue/green deployments
	- Progressive delivery

You can optionally integrate argo rollouts with an ingress controller and service mesh to perform canary and blue/green deployments. 

While Kubernetes provides `RollingUpdate` in its native Deployment Object, users are unable to control the speed, traffic flow, readiness probes, external metrics, or abort/rollback the deployment. Therefore, in large-scale high-volume production environments it's recommended to use an Argo rollout. 

