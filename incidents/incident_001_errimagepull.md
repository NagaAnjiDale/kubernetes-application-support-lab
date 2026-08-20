# Incident 001 - ErrImagePull / ImagePullBackOff

## Issue

A new pod failed to start during a deployment rollout.

## Symptoms

The new pod showed:

- `ErrImagePull`
- `ImagePullBackOff`

## Investigation

Checked the pod using:

```bash
kubectl describe pod <pod-name> -n support-lab