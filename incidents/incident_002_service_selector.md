# Incident 002 - Service Selector Mismatch

## Issue

Application pods were running, but the Kubernetes Service could not route traffic to them.

## Symptoms

- Pods were `Running`
- Service had no endpoints
- HTTP request to the Service failed

## Investigation

Checked the Service:

```bash
kubectl describe service support-app-service -n support-lab