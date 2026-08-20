# Incident 001 - ErrImagePull / ImagePullBackOff

## Issue

A new pod failed to start during a Deployment rollout because Kubernetes could not pull the configured container image.

## Symptoms

The new pod showed:

- `ErrImagePull`
- `ImagePullBackOff`
- Pod remained `0/1` Ready
- Existing application pods remained running during the failed rollout

## Investigation

Checked the pod status:

```bash
kubectl get pods -n support-lab
```

The new pod showed an image pull failure.

Inspected the failed pod:

```bash
kubectl describe pod <pod-name> -n support-lab
```

The container configuration showed:

```text
Image: nginx:does-not-exist
State: Waiting
Reason: ErrImagePull
Ready: False
```

The Events section showed Kubernetes repeatedly attempting to pull the image:

```text
Pulling image "nginx:does-not-exist"
Failed to pull image "nginx:does-not-exist"
Error: ErrImagePull
Back-off pulling image "nginx:does-not-exist"
Error: ImagePullBackOff
```

The detailed error indicated that the requested image tag could not be found in the container registry.

## Root Cause

The Deployment referenced an invalid container image tag:

```text
nginx:does-not-exist
```

Because the image did not exist, the kubelet could not pull the image and start the container.

## Resolution

Rolled back the Deployment to the previous working revision:

```bash
kubectl rollout undo deployment/support-app -n support-lab
```

## Validation

Verified the pods after recovery:

```bash
kubectl get pods -n support-lab
```

The application pods returned to:

```text
READY   STATUS    RESTARTS
1/1     Running   0
1/1     Running   0
```

Verified the Deployment rollout status:

```bash
kubectl rollout status deployment/support-app -n support-lab
```

Result:

```text
deployment "support-app" successfully rolled out
```

## Troubleshooting Summary

```text
New pod failed to start
        |
        v
Checked pod status
        |
        v
ErrImagePull detected
        |
        v
Inspected pod Events
        |
        v
Invalid image tag identified
        |
        v
Rolled back Deployment
        |
        v
Validated healthy pods and rollout
```

## Key Learning

`ErrImagePull` and `ImagePullBackOff` indicate that Kubernetes is unable to retrieve the configured container image. Pod events are an important first source of diagnostic information for identifying image-name, image-tag, registry-access, or authentication problems.
