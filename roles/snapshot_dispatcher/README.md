# snapshot_dispatcher

Dispatcher role for `philip860.leapp_snapshot`.

## Purpose

This role is the main entry point for snapshot workflows before a RHEL Leapp upgrade.

It detects the platform and selects the appropriate snapshot provider.

## Supported behavior

- Honors `snapshot_skip`
- Validates `snapshot_action`
- Detects AWS EC2
- Detects LVM
- Detects VMware/KVM/bare metal indicators
- Calls the selected snapshot provider

## Variables

```yaml
snapshot_skip: false
snapshot_action: check
snapshot_provider: auto
snapshot_set_name: leapp-preupgrade
snapshot_fail_when_unsupported: false