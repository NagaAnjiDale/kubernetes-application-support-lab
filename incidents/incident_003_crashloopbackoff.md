# Incident 003 - CrashLoopBackOff

## Issue

New application pods repeatedly failed after startup.

## Symptoms

- Pods moved from `Error` to `CrashLoopBackOff`
- Restart count kept increasing

## Investigation

Checked the failed pod:

```bash
kubectl describe pod <pod-name> -n support-lab