Que-What problem Services solve and how they relate to Pods and Deployments
Answer- he core problem Services solve is the mortality of Pods.

Because Pods are designed to be disposable, their IP addresses are temporary. If an application (like a frontend) needs to communicate with a backend database, it cannot rely on the backend Pod's IP address. The moment that database Pod crashes and is recreated, its IP changes, and the frontend's connection breaks.

A Service solves this by providing a permanent, unchanging IP address and a stable DNS name (like web-app-clusterip). It acts as a static gateway that continuously tracks the shifting, temporary IPs of your healthy Pods and automatically load-balances traffic across them.

How they connect: Labels and Selectors
These three components are not physically wired together; they connect dynamically using a tagging system.

When your Deployment creates a Pod, it slaps a sticky note on it (e.g., label: app=web-app).

When you create your Service, you give it a search criteria (e.g., selector: app=web-app).

The Service constantly scans the cluster. Any time it sees a Pod with that specific sticky note, it adds that Pod's temporary IP to its active routing list.

This decoupled design is what makes Kubernetes so resilient. The Service doesn't care which specific Pods are running, or what their IPs are. As long as the Deployment keeps manufacturing Pods with the right label, the Service will find them and route traffic to them.
  

  


  
Que - Your three Service manifests with an explanation of each type
  Ans - 
  ``` apiVersion: v1
kind: Service
metadata:
  name: web-app-clusterip
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
  ```
      How it works: This is the default Service type in Kubernetes. It creates a private, internal IP address (like the 10.96.221.155 you saw earlier) that is only reachable from inside the cluster.When to use it: Use this for backend systems that should never be exposed to the public internet, such as databases (MySQL, Redis) or internal microservices.

      NodePort

``` apiVersion: v1
kind: Service
metadata:
  name: web-app-nodeport
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```
      How it works: A NodePort builds on top of a ClusterIP. In addition to the internal IP, it opens a specific, high-numbered port (between 30000-32767) directly on the underlying physical servers (the worker nodes). In the manifest above, anyone who accesses the IP address of your worker node on port 30080 will be routed to your Nginx pods.When to use it: It is primarily used for local testing (like your Docker kind setup) to quickly expose an app to your local web browser without needing a complex cloud load balancer.

  3. LoadBalancer (The Production Standard)
```
apiVersion: v1
kind: Service
metadata:
  name: web-app-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
```
      How it works: A LoadBalancer builds on top of both a NodePort and a ClusterIP. When you apply this manifest, Kubernetes automatically sends an API request to your cloud provider (like AWS, Google Cloud, or Azure) instructing it to provision a physical, external load balancer. This external load balancer receives a public IP address and forwards internet traffic down into your cluster.When to use it: This is the standard way to expose a frontend web application to the public internet in a production cloud environment.

  
Que -The difference between ClusterIP, NodePort, and LoadBalancer
ClusterIP - Backend databases and internal microservices that must remain hidden from the outside world.
Nodeport - Local development (kind), quick testing, or bare-metal clusters without cloud integration.
Loadbalancer -Production frontend web applications running on a cloud provider (AWS, GCP, Azure).
  
  
  
How Kubernetes DNS works for service discovery
  At the heart of Kubernetes service discovery is a built-in background component called CoreDNS.

Whenever you create a new Service, Kubernetes automatically tells CoreDNS to create a custom DNS record for it. This means your applications never need to know the actual IP addresses of the services they are talking to—they only need to know the Service's name.
  
Que - What Endpoints are and how to inspect them
An Endpoint (specifically, the Endpoints resource) is the dynamically updated list of IP addresses and ports for all the healthy Pods that a Service is currently routing traffic to.

While a Service provides the stable "front door" IP, it relies entirely on its corresponding Endpoints object to know exactly where the "back door" workers (Pods) are located at any given millisecond.

When you create a Service with a selector (like app: web-app), Kubernetes automatically generates an Endpoints resource with the exact same name as the Service. It continuously monitors the cluster and instantly adds or removes Pod IPs as they crash, start up, or get deleted.

commands -  
kubectl get endpoints web-app-clusterip
kubectl describe endpoints web-app-clusterip
kubectl describe service web-app-clusterip
