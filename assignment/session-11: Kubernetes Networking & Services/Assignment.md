Session 11: Kubernetes Networking & Services

Task 1: Kubernetes Services

Deploy and demonstrate all 5 Service types:
ClusterIP
NodePort
LoadBalancer
ExternalName
Headless
For each Service:
Create the required YAML
Deploy the application
Verify the Service
Test connectivity
Capture output
Add output/screenshots to README.md
Task 2: Kubernetes Object Comparison

Create a README.md explaining:

Deployment vs ReplicaSet
Explain:
Purpose
Pod management
Scaling
Rolling updates
Relationship between Deployment and ReplicaSet
Deployment vs DaemonSet vs StatefulSet
Compare:
Use cases
Pod creation
Scaling
Networking
Storage
Examples
ReplicaSet vs Service
Explain:
ReplicaSet responsibility
Service responsibility
Why a Service is required
How traffic reaches Pods
Task 3: FQDN
Create:
fqdn/
└── README.md
Document:
What is FQDN?
Kubernetes Service DNS
Kubernetes DNS naming convention
Namespace-based DNS
Pod-to-Service communication
Examples of Kubernetes FQDNs
Task 4: CoreDNS
Research and document:
What is CoreDNS?
Why Kubernetes uses CoreDNS
How Service discovery works
How DNS queries are resolved
CoreDNS configuration
How to troubleshoot DNS issues
Deliverables
Service YAML files
Comparison documentation
fqdn/README.md
coredns/README.md
Screenshots/output

