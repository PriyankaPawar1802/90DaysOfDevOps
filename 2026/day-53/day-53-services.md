Services :

Think of your Kubernetes cluster as a Customer Support Center.

The Service IP (10.96...) is the company's 1-800 toll-free number. It is permanent. It gets printed on business cards. It never changes.

The Pod IPs (10.244...) are the personal desk extensions of the employees working today. They are temporary. Employees shift desks, go home sick, or get replaced.

Why we cannot just use Pod IPs:
Imagine an internal app in your company needs to talk to your Nginx web app. If you tell that app to connect directly to a Pod IP (10.244.0.10), it will work for a while.

But Pods are disposable. If the physical server hosting that Pod restarts, or the Pod crashes, Kubernetes deletes it and spins up a replacement. That replacement will get a brand new IP (like 10.244.0.19). Now, your internal app is sending traffic to a dead IP address, and your system is broken.

The Solution:
To prevent these dead connections, Kubernetes forces a separation:

Apps only ever talk to the permanent Service IP (10.96.221.155).

The Service acts as the automated switchboard operator. It keeps a real-time list of which employees (Pods) are currently clocked in.

When a request comes in, the Service instantly forwards it to a healthy Pod IP.

The IPs look completely different simply so the Kubernetes system can instantly identify which addresses are permanent switchboards (the 10.96 network) and which are temporary, disposable workers (the 10.244 network).





How this works under the hood:
When you created your Service with the selector app: web-app, Kubernetes didn't just create the Service. It automatically generated a hidden background resource called an Endpoint (with the exact same name as the Service).

This Endpoint constantly watches your cluster. If one of your Pods crashes and restarts with a new 10.244.x.x IP address, the Endpoint instantly updates its internal list. This dynamic list is what the Service (10.96.221.155) uses to know exactly where to forward your traffic at any given millisecond.





Kubernetes intentionally operates two completely separate IP address ranges (subnets) to separate temporary Pods from permanent Services.

The Pod Network (10.244.x.x): Your kind cluster uses a networking plugin (Kindnet) that reserves the 10.244.0.0/16 block specifically for Pods. Every time a pod is created, it gets a unique, temporary IP from this pool.

The Service Network (10.96.x.x): The Kubernetes control plane reserves the 10.96.0.0/12 block exclusively for Services. This is why your web-app-clusterip received 10.96.221.155. These are "virtual" IPs that do not belong to a physical network interface.

How they work together:
When traffic hits the virtual Service IP (10.96.221.155), a background component called kube-proxy intercepts the request and instantly forwards it to one of the actual Pod IPs (10.244.0.10, .11, or .12).

Because Pods can crash and their 10.244 IPs change constantly, you never configure applications to talk to Pod IPs directly. You only ever configure them to talk to the stable 10.96 Service IP, and Kubernetes handles the underlying routing automatically.





  








  
