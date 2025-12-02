# Velero Plugin for CSI - CleanStart Container

A containerized version of the Velero Plugin for CSI that enables Velero to create and restore CSI volume snapshots for persistent volumes in Kubernetes clusters.

## Overview

This container provides the **Velero Plugin for CSI** binary, packaged as a lightweight init container image. It is designed to be used as an init container in Velero deployments to make the CSI plugin available to the main Velero server, enabling volume snapshot functionality.

### What is Velero?

Velero is an open-source tool for safely backing up and restoring Kubernetes cluster resources and persistent volumes. It provides:
- **Backup and restore** of Kubernetes cluster resources
- **Disaster recovery** capabilities
- **Data migration** between clusters
- **Scheduled backups** with retention policies

### What Does This Plugin Do?

The Velero Plugin for CSI extends Velero's capabilities to work with **Container Storage Interface (CSI)** drivers. This plugin enables:

- **Volume Snapshots**: Create CSI volume snapshots for persistent volumes during backups
- **Volume Restore**: Restore persistent volumes from CSI snapshots during restore operations
- **CSI Driver Integration**: Works with any CSI-compatible storage driver (AWS EBS, GCE PD, Azure Disk, etc.)
- **Stateful Application Backup**: Properly backup and restore stateful applications with persistent data
- **Snapshot Lifecycle Management**: Leverage CSI snapshot classes for snapshot policies

### How It Works

This container is used as an **init container** in Velero deployments:

1. **Init Container Phase**: The container runs before the main Velero container starts
2. **Plugin Copy**: It copies the Velero CSI plugin binary from `/plugins/velero-plugin-for-csi` to `/target/velero-plugin-for-csi` on a shared volume
3. **Main Container Phase**: The Velero server container starts and reads the plugin from `/plugins/` on the same shared volume
4. **Result**: Velero can now create and restore CSI volume snapshots for persistent volumes

## 🖼️ Image Information

**Image Name:** `cleanstart/velero-plugin-for-csi:latest-dev`

**Image Details:**
- **Base Architecture**: `amd64`
- **OS**: `linux`
- **Entrypoint**: `/bin/cp-plugin /plugins/velero-plugin-for-csi /target/velero-plugin-for-csi`
- **User**: `clnstrt` (non-root, UID 1000)
- **SSL Certificates**: Pre-configured at `/etc/ssl/certs/ca-certificates.crt`

**Security Features:**
- Runs as non-root user (`clnstrt`, UID 1000)
- Minimal attack surface (single-purpose container)
- SSL/TLS certificates pre-configured
- Security context with dropped capabilities

## Use Cases

1. **Stateful Application Backup**: Backup stateful applications (databases, message queues) with their persistent volumes
2. **Volume Snapshot Management**: Create and manage CSI volume snapshots for persistent volumes
3. **Disaster Recovery**: Restore entire applications including their persistent data from snapshots
4. **Cluster Migration**: Migrate stateful workloads between clusters using volume snapshots
5. **Point-in-Time Recovery**: Restore applications to specific points in time using volume snapshots
6. **Multi-Cloud Backup**: Work with CSI drivers across different cloud providers (AWS, GCP, Azure, etc.)

## Prerequisites

To use this plugin in production, you need:

1. **Kubernetes Cluster** with CSI support (Kubernetes 1.17+)
2. **CSI Driver**: A CSI-compatible storage driver installed in your cluster (e.g., AWS EBS CSI, GCE PD CSI, Azure Disk CSI)
3. **VolumeSnapshotClass**: A VolumeSnapshotClass resource configured for your CSI driver
4. **Velero**: Velero server installed in your cluster

## Quick Start

Test the plugin functionality without requiring CSI drivers or Helm by using the test deployment in the `kubernetes/` directory. This deploys a test environment that verifies the plugin binary is correctly copied and available.

For detailed testing instructions and production deployment steps, see `kubernetes/README.md`.

## 🔧 Technical Details

### Container Entrypoint

The container uses a custom entrypoint script (`/bin/cp-plugin`) that:
- Copies the plugin binary from the source location (`/plugins/velero-plugin-for-csi`)
- Places it in the target location (`/target/velero-plugin-for-csi`)
- Ensures proper permissions and ownership

### Volume Requirements

- **Init Container**: Mount shared volume at `/target` (write access)
- **Velero Container**: Mount same volume at `/plugins` (read access)
- **Volume Type**: `emptyDir` for ephemeral storage, or persistent volume for persistence

### Integration Pattern

The plugin is integrated by adding it as an init container alongside the main Velero container. The init container copies the plugin binary to a shared volume, which is then mounted in the Velero container at the `/plugins` directory. This allows Velero to discover and load the CSI plugin automatically.

### CSI Driver Compatibility

This plugin works with any CSI-compatible storage driver, including:
- **AWS EBS CSI Driver**: For Amazon Elastic Block Store volumes
- **GCE PD CSI Driver**: For Google Compute Engine Persistent Disks
- **Azure Disk CSI Driver**: For Azure Managed Disks
- **OpenStack Cinder CSI**: For OpenStack Cinder volumes
- **vSphere CSI Driver**: For vSphere volumes
- **Other CSI Drivers**: Any driver that implements the CSI specification

## 📚 Documentation

- **Kubernetes Deployment Guide**: See `kubernetes/README.md` for detailed testing and production deployment instructions
- **Integration Examples**: See `kubernetes/deployment.yaml` for complete YAML examples

This container is designed to work with Velero and requires a CSI driver to be installed in your cluster. See `kubernetes/README.md` for production deployment instructions.

