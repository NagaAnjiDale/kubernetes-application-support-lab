# Incident 002 - Service Selector Mismatch

## Issue

Application pods were running successfully, but the Kubernetes Service could not route traffic to them.

## Symptoms

- Application pods remained `Running` and `Ready`
- Service had no endpoints
- HTTP requests through the Service failed
- The application itself was still running inside the pods

## Investigation

Checked the application pods:

```bash
kubectl get pods -n support-lab
```

The pods were running successfully, indicating that the issue was not with the application containers.

Checked the Service configuration:

```bash
kubectl describe service support-app-service -n support-lab
```

The Service showed:

```text
Selector: app=wrong-app
Endpoints:
```

The empty `Endpoints` field indicated that the Service had no backend pods available for routing traffic.

Checked the application pod labels and found that the pods used:

```text
app=support-app
```

The Service selector expected:

```text
app=wrong-app
```

Because the Service selector did not match the pod labels, Kubernetes could not associate the application pods with the Service.

## Connectivity Test

Tested the application through the Service from inside the cluster:

```bash
kubectl run curl-test \
  --rm -it \
  --restart=Never \
  --image=curlimages/curl \
  -n support-lab \
  -- curl --max-time 5 http://support-app-service
```

The request failed:

```text
curl: (7) Failed to connect to support-app-service:80
```

## Root Cause

The Kubernetes Service had an incorrect selector:

```text
app=wrong-app
```

while the application pods were labeled:

```text
app=support-app
```

The selector mismatch resulted in the Service having no endpoints.

## Resolution

Corrected the Service selector:

```bash
kubectl patch service support-app-service -n support-lab \
-p '{"spec":{"selector":{"app":"support-app"}}}'
```

## Validation

Checked the Service again:

```bash
kubectl describe service support-app-service -n support-lab
```

The Service now showed the correct selector and backend endpoints:

```text
Selector:  app=support-app
Endpoints: <pod-ip>:80,<pod-ip>:80
```

Repeated the HTTP connectivity test:

```bash
kubectl run curl-test \
  --rm -it \
  --restart=Never \
  --image=curlimages/curl \
  -n support-lab \
  -- curl --max-time 5 http://support-app-service
```

The request successfully returned the Nginx welcome page, confirming that Service-to-pod connectivity was restored.

## Troubleshooting Summary

```text
Application unreachable through Service
        |
        v
Checked pods
        |
        v
Pods Running and Ready
        |
        v
Checked Service
        |
        v
No endpoints found
        |
        v
Compared selector with pod labels
        |
        v
Selector mismatch identified
        |
        v
Corrected Service selector
        |
        v
Endpoints restored
        |
        v
Validated HTTP connectivity
```

## Key Learning

A pod can be completely healthy while the application remains inaccessible through its Service.

When pods are running but Service connectivity fails, checking the Service selector, pod labels, and endpoints helps determine whether Kubernetes has correctly associated the Service with its backend pods.
