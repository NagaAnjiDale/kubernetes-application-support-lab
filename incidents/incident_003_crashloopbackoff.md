# Incident 003 - CrashLoopBackOff

## Issue

New application pods repeatedly terminated after startup, causing Kubernetes to restart the containers and eventually place the pods in `CrashLoopBackOff`.

## Symptoms

- New pods initially showed `Error`
- Pods later entered `CrashLoopBackOff`
- Restart count continued increasing
- Containers were not reaching the `Ready` state
- An existing healthy pod remained available during the failed rollout

## Investigation

Checked the pod status:

```bash
kubectl get pods -n support-lab
```

The affected pods showed:

```text
READY   STATUS             RESTARTS
0/1     CrashLoopBackOff   5
0/1     CrashLoopBackOff   5
```

Inspected one of the failed pods:

```bash
kubectl describe pod <pod-name> -n support-lab
```

The container state showed:

```text
State:       Terminated
Reason:      Error
Exit Code:   1
Ready:       False
```

The configured container command was:

```text
/bin/sh
-c
exit 1
```

The pod Events also showed Kubernetes repeatedly restarting the failed container:

```text
Container created
Container started
Back-off restarting failed container
```

Checked the logs from the previous container instance:

```bash
kubectl logs <pod-name> -n support-lab --previous
```

No application logs were returned because the configured command exited immediately without writing output.

## Root Cause

The Deployment had been configured with a command that intentionally terminated the container with exit code `1`:

```text
/bin/sh -c exit 1
```

Because the container exited immediately after starting, Kubernetes repeatedly restarted it. Repeated failures resulted in the pod entering `CrashLoopBackOff`.

## Recovery Investigation

An attempt was made to restore the healthy Deployment configuration using:

```bash
kubectl apply -f manifests/deployment.yaml
```

However, the rollout did not complete successfully and continued waiting for the old failing replicas.

Deleting the failed pods did not resolve the problem because the Deployment's live pod template still contained the failing command. The ReplicaSet therefore recreated new pods with the same configuration.

The live Deployment configuration was checked using:

```bash
kubectl get deployment support-app -n support-lab -o yaml
```

The failing command was still present:

```text
command:
- /bin/sh
- -c
- exit 1
```

This confirmed that deleting individual pods would not fix the underlying Deployment configuration.

## Resolution

Removed the failing command from the Deployment's pod template:

```bash
kubectl patch deployment support-app -n support-lab \
  --type='json' \
  -p='[{"op":"remove","path":"/spec/template/spec/containers/0/command"}]'
```

Kubernetes then created replacement pods using the corrected container configuration.

## Validation

Checked the application pods:

```bash
kubectl get pods -n support-lab
```

The replacement pods returned to a healthy state:

```text
READY   STATUS    RESTARTS
1/1     Running   0
1/1     Running   0
```

Verified the overall application resources:

```bash
kubectl get deployment,pods,svc -n support-lab
```

The Deployment showed the required replicas available and the application pods were running successfully.

## Troubleshooting Summary

```text
New pods repeatedly failed
        |
        v
Checked pod status
        |
        v
CrashLoopBackOff detected
        |
        v
Inspected container state and Events
        |
        v
Exit Code 1 identified
        |
        v
Checked previous container logs
        |
        v
Failing command identified
        |
        v
Attempted configuration recovery
        |
        v
Failed pods were recreated
        |
        v
Checked live Deployment template
        |
        v
Failing command still present
        |
        v
Corrected Deployment configuration
        |
        v
Validated healthy replacement pods
```

## Key Learning

`CrashLoopBackOff` is not the root cause itself. It indicates that a container is repeatedly starting and failing while Kubernetes progressively delays additional restart attempts.

A useful investigation sequence is to check the pod status, inspect the container's current and previous states, review its exit code and Events, and examine current or previous container logs.

Deleting a failing pod is not a permanent resolution when the underlying Deployment configuration is incorrect, because the controller will recreate the pod using the same faulty configuration.
