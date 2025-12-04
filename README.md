# Velero Plugin for CSI - CleanStart Container

A containerized version of the Velero Plugin for CSI that enables Velero to create and restore CSI volume snapshots for persistent volumes in Kubernetes clusters. The CleanStart Velero-Plugin-For-CSI image provides a production-ready, security-hardened container optimized for enterprise environments. Built on a minimal base OS with comprehensive security hardening, this image delivers reliable application execution with advanced security features.

**📌 CleanStart Foundation:** Security-hardened, minimal base OS designed for enterprise containerized environments.

**Image Path:** `ghcr.io/cleanstart-containers/velero-plugin-for-csi`

**Registry:** CleanStart Registry

---

## Overview

This container provides the **Velero Plugin for CSI** binary, packaged as a lightweight init container image. It is designed to be used as an init container in Velero deployments to make the CSI plugin available to the main Velero server, enabling volume snapshot functionality. This Velero-Plugin-For-CSI container is part of the CleanStart application suite, featuring enterprise-grade security hardening, automated vulnerability management, and compliance with industry standards.

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

---

## About CleanStart

CleanStart is a comprehensive container registry providing security-hardened, enterprise-ready container images. Our images are designed with security-first principles, featuring minimal attack surfaces, regular security updates, and compliance with industry standards.

### About CleanStart Images

CleanStart images are built on secure, minimal base operating systems and optimized for production environments. Each image undergoes rigorous security testing, vulnerability scanning, and compliance validation to ensure enterprise-grade security and reliability.

---

## 🖼️ Image Information

**Image Name:** `ghcr.io/cleanstart-containers/velero-plugin-for-csi:latest-dev`

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

---

## Key Features

- **Security-First Design**: Built with minimal attack surfaces and security hardening
- **Enterprise Compliance**: Meets industry standards including FIPS, STIG, and CIS benchmarks
- **Regular Updates**: Automated security patches and vulnerability management
- **Multi-Architecture Support**: Available for AMD64 and ARM64 architectures
- **Production Ready**: Optimized for enterprise deployment and scaling
- **Comprehensive Documentation**: Detailed guides and best practices for each image
- Lightweight init container for Velero CSI plugin integration
- CSI driver compatibility across multiple cloud providers
- Automated plugin binary deployment
- Non-root execution for enhanced security

---

## Use Cases

Typical scenarios where this container excels:

1. **Stateful Application Backup**: Backup stateful applications (databases, message queues) with their persistent volumes
2. **Volume Snapshot Management**: Create and manage CSI volume snapshots for persistent volumes
3. **Disaster Recovery**: Restore entire applications including their persistent data from snapshots
4. **Cluster Migration**: Migrate stateful workloads between clusters using volume snapshots
5. **Point-in-Time Recovery**: Restore applications to specific points in time using volume snapshots
6. **Multi-Cloud Backup**: Work with CSI drivers across different cloud providers (AWS, GCP, Azure, etc.)

---

## Prerequisites

To use this plugin in production, you need:

1. **Kubernetes Cluster** with CSI support (Kubernetes 1.17+)
2. **CSI Driver**: A CSI-compatible storage driver installed in your cluster (e.g., AWS EBS CSI, GCE PD CSI, Azure Disk CSI)
3. **VolumeSnapshotClass**: A VolumeSnapshotClass resource configured for your CSI driver
4. **Velero**: Velero server installed in your cluster

---

## Quick Start

### Pull Commands
```bash
docker pull ghcr.io/cleanstart-containers/velero-plugin-for-csi:latest
docker pull ghcr.io/cleanstart-containers/velero-plugin-for-csi:latest-dev
```

### Run Commands

Basic test:
```bash
docker run -it --name velero-plugin-for-csi-test ghcr.io/cleanstart-containers/velero-plugin-for-csi:latest-dev
```

Production deployment:
```bash
docker run -d --name velero-plugin-for-csi-prod \
  --read-only \
  --security-opt=no-new-privileges \
  --user 1000:1000 \
  ghcr.io/cleanstart-containers/velero-plugin-for-csi:latest
```

### Testing

Test the plugin functionality without requiring CSI drivers or Helm by using the test deployment in the `kubernetes/` directory. This deploys a test environment that verifies the plugin binary is correctly copied and available.

For detailed testing instructions and production deployment steps, see `kubernetes/README.md`.

---

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

---

## Architecture Support

CleanStart images support multiple architectures to ensure compatibility across different deployment environments:

- **AMD64**: Intel and AMD x86-64 processors
- **ARM64**: ARM-based processors including Apple Silicon and ARM servers

### Architecture-based Pull Commands
```bash
docker pull --platform linux/amd64 ghcr.io/cleanstart-containers/velero-plugin-for-csi:latest
docker pull --platform linux/arm64 ghcr.io/cleanstart-containers/velero-plugin-for-csi:latest
```

---

## 📚 Documentation

- **Kubernetes Deployment Guide**: See `kubernetes/README.md` for detailed testing and production deployment instructions
- **Integration Examples**: See `kubernetes/deployment.yaml` for complete YAML examples

This container is designed to work with Velero and requires a CSI driver to be installed in your cluster. See `kubernetes/README.md` for production deployment instructions.

---

## Resources

- **Official Documentation:** https://velero.io/docs/
- **Velero CSI Plugin Documentation:** https://github.com/vmware-tanzu/velero-plugin-for-csi
- **Provenance / SBOM / Signature:** https://images.cleanstart.com/images/velero-plugin-for-csi
- **Docker Hub:** https://hub.docker.com/r/cleanstart/velero-plugin-for-csi
- **CleanStart All Images:** https://images.cleanstart.com
- **CleanStart Community Images:** https://hub.docker.com/u/cleanstart

---

## Disclaimer & License

### Disclaimer

**Disclaimer:** This documentation is provided for informational purposes only. Users are responsible for ensuring compliance with applicable laws, regulations, and security requirements. CleanStart makes no warranties regarding the suitability of these images for specific use cases or environments.

### License

Apache-2.0

---

## Vulnerability Disclaimer

CleanStart offers Docker images that include third-party open-source libraries and packages maintained by independent contributors. While CleanStart maintains these images and applies industry-standard security practices, it cannot guarantee the security or integrity of upstream components beyond its control.

Users acknowledge and agree that open-source software may contain undiscovered vulnerabilities or introduce new risks through updates. CleanStart shall not be liable for security issues originating from third-party libraries, including but not limited to zero-day exploits, supply chain attacks, or contributor-introduced risks.

**Security remains a shared responsibility:** CleanStart provides updated images and guidance where possible, while users are responsible for evaluating deployments and implementing appropriate controls.
