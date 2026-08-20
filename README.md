# Kubernetes Application Support Lab

Hands-on lab demonstrating Kubernetes application troubleshooting and production support scenarios.

## Environment

- Kubernetes: k3s
- Application: Nginx
- Namespace: `support-lab`
- Workload: Deployment with 2 replicas
- Service: ClusterIP

## Scenarios

### Incident 001 - ErrImagePull / ImagePullBackOff
Troubleshooting a failed deployment caused by an invalid container image.

### Incident 002 - Service Selector Mismatch
Troubleshooting healthy application pods that are unreachable through the Kubernetes Service.

### Incident 003 - CrashLoopBackOff
Troubleshooting containers that repeatedly terminate and restart.

## Skills Demonstrated

- Pod and Deployment troubleshooting
- `kubectl get`, `describe`, and `logs`
- Kubernetes Events analysis
- Container image troubleshooting
- CrashLoopBackOff investigation
- Service and endpoint troubleshooting
- Label and selector validation
- Deployment recovery and validation