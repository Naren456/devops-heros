Session 13: Kubernetes Storage, HPA & Probes

Task 1: Kubernetes Volumes
Create:
01-kubernetes-volumes/
└── README.md
Document what you learned about:
emptyDir
hostPath
PersistentVolume
PersistentVolumeClaim
StorageClass
Dynamic provisioning
Include practical examples wherever possible.

Task 2: HPA Hands-on
Use hpa.yml and perform the following:
Deploy the application.
Configure HPA.
Verify HPA.
Deploy a load generator.
Increase application load.
Observe CPU utilization.
Observe Pod scaling.
Capture the output.
Add the output/screenshots to README.md.
Useful commands should include:

kubectl get hpa
kubectl get pods
kubectl top pods
kubectl describe hpa

Task 3: Mini Project

Complete the mini project provided for Session 13.

Deliverables
Volume documentation
HPA YAML
Load generator
HPA output
Screenshots
Mini-project implementation
README documentation


