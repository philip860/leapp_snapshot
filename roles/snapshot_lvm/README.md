# philip860.leapp_snapshot

Snapshot dispatcher and provider collection for Red Hat Enterprise Linux Leapp upgrade workflows.

## Purpose

`philip860.leapp_snapshot` provides a common snapshot framework for RHEL hosts before running Leapp upgrades.

It is designed to support hosts running across multiple platforms, including:

- LVM-backed RHEL systems
- AWS EC2
- VMware
- Azure
- Google Cloud
- KVM/libvirt
- Bare metal systems

The collection allows a Leapp workflow to use one common snapshot entry point while the dispatcher selects the appropriate snapshot provider.

## Supported Providers

| Provider | Status |
|---|---|
| LVM | Supported |
| AWS EBS | Planned |
| VMware Snapshot API | Planned |
| Azure Managed Disk | Planned |
| Google Cloud Disk | Planned |
| KVM/libvirt | Planned |
| Bare Metal | Planned |

## Snapshot Flow

```text
snapshot_dispatcher
        |
        +--> snapshot_lvm
        |
        +--> snapshot_aws
        |
        +--> snapshot_vmware
        |
        +--> snapshot_azure
        |
        +--> snapshot_gcp
        |
        +--> snapshot_kvm
        |
        +--> snapshot_baremetal