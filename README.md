# Kubernetes Application Support Lab

A hands-on Kubernetes lab demonstrating application troubleshooting, incident investigation, root cause identification, and service recovery in a containerized environment.

The project simulates common production support scenarios and documents the troubleshooting process used to identify and resolve each issue.

## Environment

- Kubernetes: k3s
- Application: Nginx
- Namespace: `support-lab`
- Workload: Deployment with 2 replicas
- Service: ClusterIP

## Repository Structure

```text
kubernetes-application-support-lab/
├── README.md
├── manifests/
│   ├── deployment.yaml
│   └── service.yaml
├── incidents/
│   ├── incident_001_errimagepull.md
│   ├── incident_002_service_selector.md
│   └── incident_003_crashloopbackoff.md
└── screenshots/
```

## Application Architecture

```text
Client / Test Pod
       |
       v
ClusterIP Service
support-app-service
       |
       | app=support-app
       v
+-------------------+
|   Deployment      |
|   support-app     |
+-------------------+
       |
       +------------------+
       |                  |
       v                  v
 support-app Pod     support-app Pod
     Nginx               Nginx
```

## Deployment

The application can be deployed using the manifests in the `manifests` directory:

```bash
kubectl apply -f manifests/deployment.yaml
kubectl apply -f manifests/service.yaml
```

Verify the resources:

```bash
kubectl get deployment,pods,svc -n support-lab
```

## Incident Scenarios

### Incident 001 - ErrImagePull / ImagePullBackOff

A Deployment rollout failed because Kubernetes could not pull the configured container image.

The investigation included:

- Checking pod status
- Inspecting pod events
- Identifying the invalid image tag
- Recovering the Deployment
- Validating application availability after recovery

See: `incidents/incident_001_errimagepull.md`

### Incident 002 - Service Selector Mismatch

Application pods were running successfully, but the application was unreachable through the Kubernetes Service.

The investigation included:

- Checking pod health
- Inspecting the Service configuration
- Checking Service endpoints
- Comparing Service selectors with pod labels
- Correcting the selector
- Validating connectivity after recovery

See: `incidents/incident_002_service_selector.md`

### Incident 003 - CrashLoopBackOff

Application containers repeatedly terminated after startup, causing Kubernetes to restart them and eventually report `CrashLoopBackOff`.

The investigation included:

- Checking pod status and restart count
- Inspecting container state and exit code
- Reviewing pod events
- Checking previous container logs
- Identifying the failing container command
- Restoring the Deployment
- Validating healthy replacement pods

See: `incidents/incident_003_crashloopbackoff.md`

## Troubleshooting Workflow

The incidents in this lab follow a structured production support approach:

```text
Identify the issue
        |
        v
Check resource status
        |
        v
Inspect pod / service details
        |
        v
Review events and logs
        |
        v
Identify root cause
        |
        v
Apply corrective action
        |
        v
Validate recovery
```

Common commands used during investigation include:

```bash
kubectl get pods -n support-lab
kubectl describe pod <pod-name> -n support-lab
kubectl logs <pod-name> -n support-lab
kubectl logs <pod-name> -n support-lab --previous
kubectl describe service support-app-service -n support-lab
kubectl get endpointslices -n support-lab
kubectl rollout status deployment/support-app -n support-lab
```

## Skills Demonstrated

- Kubernetes application troubleshooting
- Pod and Deployment investigation
- `ErrImagePull` and `ImagePullBackOff` troubleshooting
- `CrashLoopBackOff` investigation
- Container state and exit-code analysis
- Kubernetes Events analysis
- Application log investigation
- Service and endpoint troubleshooting
- Pod label and Service selector validation
- Deployment rollout and recovery
- Post-incident validation
- Production support troubleshooting workflow
