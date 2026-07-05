## `roles/snapshot_lvm/README.md`

```markdown
# snapshot_lvm

LVM snapshot provider role for `philip860.leapp_snapshot`.

## Purpose

This role performs LVM snapshot readiness checks and delegates snapshot create, remove, and revert actions to `infra.lvm_snapshots`.

## Supported actions

- check
- create
- remove
- revert

## Variables

```yaml
snapshot_action: create
snapshot_set_name: leapp-preupgrade

snapshot_lvm_volumes:
  - vg: VolGroup00
    lv: rootVol
    size: 4G