# CleanStart MetalLB Speaker Kubernetes Deployment

This directory contains Kubernetes deployment configuration for testing the `cleanstart/metallb-speaker:latest-dev` Docker image.

## Prerequisites

- Kubernetes cluster access with `kubectl` configured
- Docker image `cleanstart/metallb-speaker:latest-dev` available in your cluster's image registry or locally built

## Image Information

The deployment uses the following image configuration:
- **Image**: `cleanstart/metallb-speaker:latest-dev`
- **Entrypoint**: `/usr/bin/metallb-speaker`
- **User**: `clnstrt` (UID 1000)
- **Architecture**: `amd64`
- **OS**: `linux`

## Deployment Steps

### Step 1: Install MetalLB CRDs

MetalLB Custom Resource Definitions (CRDs) are required for the speaker to function properly:

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/main/config/manifests/metallb-native.yaml
```

Wait for CRDs to be established:

```bash
kubectl wait --for condition=established --timeout=60s crd/bfdprofiles.metallb.io
kubectl wait --for condition=established --timeout=60s crd/bgppeers.metallb.io
kubectl wait --for condition=established --timeout=60s crd/bgpadvertisements.metallb.io
kubectl wait --for condition=established --timeout=60s crd/ipaddresspools.metallb.io
kubectl wait --for condition=established --timeout=60s crd/l2advertisements.metallb.io
```

### Step 2: Deploy to Kubernetes

Apply the deployment configuration:

```bash
kubectl apply -f deployment.yaml
```

### Step 3: Verify Deployment

Check the deployment status:

```bash
kubectl get deployment metallb-speaker -w
kubectl get pods -l app=metallb-speaker -w
```

**Expected Result:**
- Deployment shows `READY 1/1` and `AVAILABLE 1`
- Pod shows `Running` status

### Step 4: Check Pod Logs

Monitor the pod logs to verify the image functionality:

```bash
kubectl logs -f deployment/metallb-speaker
```

**Expected Log Output:**
- ✅ `"MetalLB speaker starting version 0.15.2"` - Image starts successfully
- ✅ `"Starting Manager"` - Manager initializes successfully
- ✅ `"Starting Controller"` - Controllers start successfully
- ⚠️ `"failed to create ARP responder"` - Expected without host networking (non-fatal)

## Useful Commands

```bash
# View all resources
kubectl get all -l app=metallb-speaker

# View events
kubectl get events --sort-by='.lastTimestamp' | grep metallb-speaker
```

## Cleanup

To remove the deployment:

```bash
# Remove deployment
kubectl delete -f deployment.yaml

# Optional: Remove MetalLB CRDs
kubectl delete -f https://raw.githubusercontent.com/metallb/metallb/main/config/manifests/metallb-native.yaml
```

## Notes

- This deployment is configured for **testing purposes** to verify image functionality
- For production use, MetalLB speaker typically runs as a **DaemonSet** with host networking enabled
- The deployment uses non-root user (UID 1000) for security
- Resource limits are set to prevent excessive resource consumption during testing
- **CAP_NET_RAW** capability is included for ARP operations
