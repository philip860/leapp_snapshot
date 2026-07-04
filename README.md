# community.snapshot

Cross-platform snapshot automation collection.

## Purpose

community.snapshot provides a common snapshot framework for RHEL hosts.

It supports snapshot workflows before operations such as:

- RHEL Leapp upgrades
- patching
- maintenance windows
- configuration changes


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
        |
        +--> snapshot_lvm
        |
        +--> snapshot_aws
        |
        +--> snapshot_vmware
        |
        +--> snapshot_kvm
        |
        +--> snapshot_baremetal
