# Velero Plugin for CSI - Kubernetes Deployment Guide

Complete Kubernetes deployment guide for testing the CleanStart Velero Plugin for CSI container. This plugin enables Velero to backup and restore CSI volume snapshots.

## 📁 Files

- `deployment.yaml` - Complete deployment manifest (Namespace, ServiceAccount, Deployment, Service, and test Pod)
- `README.md` - This documentation

## 🖼️ Image Details

**Image:** `cleanstart/velero-plugin-for-csi:latest-dev`

**Key Features:**
- **Entrypoint:** `/bin/cp-plugin /plugins/velero-plugin-for-csi /target/velero-plugin-for-csi`
- **Function:** Copies the Velero CSI plugin binary from `/plugins/` to `/target/`
- **User:** `clnstrt` (non-root, UID 1000)
- **Architecture:** `amd64`
- **OS:** `linux`
- **SSL Certificates:** Pre-configured at `/etc/ssl/certs/ca-certificates.crt`

## 🎯 Purpose

The Velero Plugin for CSI is used as an **init container** in Velero deployments. It copies the plugin binary to a shared volume that the main Velero container can access. This allows Velero to create and restore CSI volume snapshots for persistent volumes.

## 🚀 Complete Deployment Steps

### Prerequisites

1. **Kubernetes cluster** (Kind, minikube, k3s, GKE, EKS, AKS, or any other)
2. **kubectl** installed and configured to access your cluster

### Step 1: Verify Kubernetes Cluster

```bash
# Check cluster connectivity
kubectl cluster-info

# Verify nodes are ready
kubectl get nodes
```

### Step 2: Deploy the Test Deployment

> **Note:** If you have previously deployed the AWS plugin test in the same cluster, make sure to clean up any conflicting resources first. The CSI plugin uses a separate namespace (`velero-plugin-csi-test`) to avoid conflicts.

```bash
# Apply the deployment manifest
kubectl apply -f deployment.yaml
```

**Expected Output:**
```
namespace/velero-plugin-csi-test created
serviceaccount/velero-plugin-csi-test created
deployment.apps/velero-plugin-csi-test created
service/velero-plugin-csi-test created
pod/velero-plugin-csi-manual-test created
```

### Step 3: Verify Deployment Status

```bash
# Check if the deployment is running
kubectl get deployment -n velero-plugin-csi-test -w

# Check pod status
kubectl get pods -n velero-plugin-csi-test -w
```

**Expected Output:**
```
NAME                        READY   STATUS    RESTARTS   AGE
velero-plugin-csi-test-xxxxx   1/1     Running   0          30s
velero-plugin-csi-manual-test   0/1     Completed   0          15s
```

### Step 4: Verify Plugin Copy Functionality

#### Option A: Check Deployment Pod Logs

```bash
# Get the deployment pod name (exclude the manual test pod by filtering for Running status)
POD_NAME=$(kubectl get pods -n velero-plugin-csi-test -l app=velero-plugin-for-csi --field-selector=status.phase=Running -o jsonpath='{.items[0].metadata.name}')

# Check the init container logs (should show successful copy)
kubectl logs $POD_NAME -n velero-plugin-csi-test -c velero-plugin-for-csi
```
**Expected Output from Init Container:**
```
Copying /plugins/velero-plugin-for-csi to /target/velero-plugin-for-csi ...  done.
```

```bash
# Check the main container logs (should show verification)
kubectl logs $POD_NAME -n velero-plugin-csi-test -c plugin-verifier
```

**Expected Output from Main Container:**
```
=== Velero Plugin for CSI Verification ===

Checking if plugin binary exists...
✅ Plugin binary found at /plugins/velero-plugin-for-csi

Plugin file details:
-rwxr-xr-x    1 1000     102        76.0M Nov 12 11:05 /plugins/velero-plugin-for-csi

File type verification:
✅ ELF 64-bit LSB executable detected (magic bytes: 7f 45 4c 46)

Plugin binary is executable: Yes

✅ Plugin verification successful!

Plugin is ready for use with Velero.
Sleeping to keep container running for inspection...
```

#### Option B: Check Manual Test Pod Logs

```bash
# Check the manual test pod logs
kubectl logs velero-plugin-csi-manual-test -n velero-plugin-csi-test -c inspector
```

**Expected Output:**
```
=== Manual Plugin Test ===

Plugin location: /plugins/velero-plugin-for-csi

✅ Plugin successfully copied!

File information:
-rwxr-xr-x    1 1000     102        76.0M Nov 12 11:05 /plugins/velero-plugin-for-csi

File type verification:
✅ ELF 64-bit LSB executable detected (magic bytes: 7f 45 4c 46)

First 100 bytes (hexdump):
00000000  7f 45 4c 46 02 01 01 00  00 00 00 00 00 00 00 00  |.ELF............|
00000010  03 00 3e 00 01 00 00 00  80 ce 48 00 00 00 00 00  |..>.......H.....|
00000020  40 00 00 00 00 00 00 00  58 7c c0 04 00 00 00 00  |@.......X|......|
00000030  00 00 00 00 40 00 38 00  0b 00 40 00 1e 00 1d 00  |....@.8...@.....|
00000040  06 00 00 00 04 00 00 00  40 00 00 00 00 00 00 00  |........@.......|
00000050  40 00 40 00 00 00 00 00  40 00 40 00 00 00 00 00  |@.@.....@.@.....|
00000060  68 02 00 00                                       |h...|
00000064

✅ Test completed successfully!
```

### Step 5: Inspect the Plugin Binary (Optional)

```bash
# Get a shell in the deployment pod (filter for Running pods to exclude the manual test pod)
POD_NAME=$(kubectl get pods -n velero-plugin-csi-test -l app=velero-plugin-for-csi --field-selector=status.phase=Running -o jsonpath='{.items[0].metadata.name}')

# Execute commands in the pod
kubectl exec -it $POD_NAME -n velero-plugin-csi-test -c plugin-verifier -- /bin/sh

# Inside the pod, you can run:
# ls -lh /plugins/velero-plugin-for-csi
# file /plugins/velero-plugin-for-csi
# /plugins/velero-plugin-for-csi --help  # If the plugin supports help
```

## 🧪 Testing the Plugin Functionality

### Test 1: Verify Plugin Binary Exists

```bash
# Check if plugin was copied successfully (using deployment name directly)
kubectl exec -n velero-plugin-csi-test deployment/velero-plugin-csi-test -c plugin-verifier -- ls -lh /plugins/velero-plugin-for-csi
```

### Test 2: Verify Plugin is Executable

```bash
# Check if plugin is executable
kubectl exec -n velero-plugin-csi-test deployment/velero-plugin-csi-test -c plugin-verifier -- test -x /plugins/velero-plugin-for-csi && echo "✅ Plugin is executable" || echo "❌ Plugin is not executable"
```

### Test 3: Check Plugin File Type

```bash
# Verify the file type (should be ELF binary - check magic bytes)
kubectl exec -n velero-plugin-csi-test deployment/velero-plugin-csi-test -c plugin-verifier -- head -c 4 /plugins/velero-plugin-for-csi | hexdump -C
```

**Note:** The output should show: `00000000  7f 45 4c 46  |.ELF|`
This confirms it's an ELF binary (7f 45 4c 46 = ELF magic bytes)

### Test 4: Verify Init Container Completed Successfully

```bash
# Check init container status (get the deployment pod first)
POD_NAME=$(kubectl get pods -n velero-plugin-csi-test -l app=velero-plugin-for-csi --field-selector=status.phase=Running -o jsonpath='{.items[0].metadata.name}')
kubectl get pod $POD_NAME -n velero-plugin-csi-test -o jsonpath='{.status.initContainerStatuses[0]}' | jq
```

## 📝 Integration with Velero (Optional - For Production Use)

The test deployment above demonstrates the plugin functionality without requiring CSI drivers or Helm. The integration examples below are for reference when you're ready to use this plugin with an actual Velero deployment in production.

### Integration Examples in deployment.yaml

All integration YAML examples are included in `deployment.yaml` for reference:

- **Lines 223-261**: Init container snippet (for modifying existing Velero deployment)
- **Lines 266-349**: Complete Velero deployment with CSI plugin
- **Lines 352-380**: Helm values.yaml configuration (commented)
- **Lines 383-398**: VolumeSnapshotLocation example

### When You're Ready to Integrate

If you have Velero installed and want to use this plugin:

1. **For existing Velero deployment**: Copy the init container configuration from `deployment.yaml` (Option 1, lines 223-261) and add it to your Velero deployment
2. **For new Velero deployment**: Use the complete deployment example from `deployment.yaml` (Option 2, lines 266-349)
3. **For Helm users**: Extract the Helm values from `deployment.yaml` (Option 3, lines 352-380) and use with the Velero Helm chart

### Key Points

- **Init container mounts at**: `/target` (where plugin copies TO)
- **Velero container mounts at**: `/plugins` (where Velero reads FROM)
- **Volume type**: Use `emptyDir: {}` for ephemeral storage
- **CSI Driver**: Requires a CSI driver to be installed in your cluster (e.g., AWS EBS CSI, GCE PD CSI, Azure Disk CSI)
- **VolumeSnapshotClass**: You'll need to configure a VolumeSnapshotClass for your CSI driver

## 🧹 Cleanup

To remove all test resources:

```bash
# Delete the namespace (this removes all resources)
kubectl delete namespace velero-plugin-csi-test

# Or delete individual resources
kubectl delete -f deployment.yaml
```

## Verification Checklist

- [ ] Deployment pod is running (1/1 Ready)
- [ ] Init container completed successfully
- [ ] Plugin binary exists at `/plugins/velero-plugin-for-csi`
- [ ] Plugin binary is executable
- [ ] Plugin binary is an ELF 64-bit executable
- [ ] Manual test pod completed successfully
- [ ] All logs show successful verification

## 🎉 Success Criteria (use case in production)

The deployment is successful when:
1. The init container copies the plugin binary without errors
2. The plugin binary is found in the shared volume
3. The plugin binary is executable and has the correct file type
4. All verification tests pass

---

This deployment is for testing purposes. In production, you would integrate this plugin as an init container in your Velero deployment to enable CSI volume snapshot functionality.

