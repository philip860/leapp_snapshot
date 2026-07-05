# snapshot_aws

AWS EC2 EBS snapshot provider role for `philip860.leapp_snapshot`.

## Purpose

This role provides AWS EC2 snapshot support before Red Hat Enterprise Linux
Leapp upgrade workflows.

The role performs AWS readiness checks, discovers the current EC2 instance,
identifies attached EBS volumes, and manages snapshots using the
`amazon.aws` Ansible collection.

This provider is normally invoked automatically by:

```yaml
philip860.leapp_snapshot.snapshot_dispatcher
```

when an AWS EC2 instance is detected.

---

## Supported actions

- check
- create
- remove
- revert

---

## Action behavior

### check

Validates AWS snapshot readiness.

The check action:

- Detects AWS EC2 using Instance Metadata Service (IMDSv2)
- Detects AWS region
- Detects EC2 instance ID
- Lists attached EBS volumes
- Validates requested volumes

Example:

```yaml
snapshot_action: check
```

---

### create

Creates EBS snapshots for the selected EC2 instance volumes.

Example:

```yaml
snapshot_action: create
snapshot_set_name: leapp-preupgrade
```

Snapshots are tagged automatically:

```yaml
SnapshotSet: leapp-preupgrade
CreatedBy: philip860.leapp_snapshot
Purpose: RHEL Leapp upgrade
InstanceId: i-xxxxxxxx
VolumeId: vol-xxxxxxxx
```

---

### remove

Removes snapshots previously created by this collection.

Example:

```yaml
snapshot_action: remove
snapshot_set_name: leapp-preupgrade
```

Snapshots are located using:

```yaml
SnapshotSet
InstanceId
```

tags.

---

### revert

AWS revert support is intentionally protected.

Unlike LVM rollback, AWS EBS rollback requires:

1. Stop EC2 instance
2. Create volumes from snapshots
3. Detach existing volumes
4. Attach restored volumes using original device names
5. Start EC2 instance

The current implementation validates available snapshots but does not
automatically replace running volumes.

---

# Required AWS Authentication

## Ansible Automation Platform

Use the built-in AAP credential type:

```text
Amazon Web Services
```

Attach this credential to the Job Template running the snapshot workflow.

Example Job Template credentials:

```yaml
credentials:
  - RHEL Machine Credential
  - AWS Credential
```

AAP automatically provides:

```bash
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN
```

to the execution environment.

No AWS keys should be stored in playbooks or variables.

---

# Required AWS IAM Permissions

The AWS credential requires permissions for EC2 snapshot management.

Minimum permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeVolumes",
        "ec2:DescribeSnapshots",
        "ec2:CreateSnapshot",
        "ec2:CreateTags",
        "ec2:DeleteSnapshot"
      ],
      "Resource": "*"
    }
  ]
}
```

---

# Variables

## Common variables

Provided by the dispatcher:

```yaml
snapshot_action: create

snapshot_set_name: leapp-preupgrade

snapshot_provider: auto
```

---

## AWS variables

Normally these are auto-detected.

```yaml
# AWS region
# blank = detect from EC2 metadata
snapshot_aws_region: ""

# EC2 instance ID
# blank = detect from EC2 metadata
snapshot_aws_instance_id: ""

# Snapshot all attached EBS volumes
snapshot_aws_snapshot_all_volumes: true
```

---

## Specific volume selection

By default, all attached EBS volumes are protected.

To snapshot specific volumes:

```yaml
snapshot_aws_volume_ids:
  - vol-xxxxxxxx
  - vol-yyyyyyyy
```

---

## Snapshot options

```yaml
snapshot_aws_description: "Leapp pre-upgrade snapshot"

snapshot_aws_wait: true

snapshot_aws_wait_timeout: 1800

snapshot_aws_delete_on_remove: true

snapshot_aws_fail_when_no_volumes: true
```

---

# Example dispatcher usage

```yaml
---
- name: Create upgrade snapshot
  hosts: all
  become: true

  roles:
    - role: philip860.leapp_snapshot.snapshot_dispatcher
```

The dispatcher automatically selects:

```text
AWS EC2 detected
        |
        v
snapshot_aws
        |
        v
EBS snapshot
```

---

# Requirements

Collections:

```yaml
amazon.aws
```

Installed automatically through:

```yaml
dependencies:
  amazon.aws: "*"
```

in `galaxy.yml`.

---

# Supported platforms

- RHEL 7
- RHEL 8
- RHEL 9
- RHEL 10

Running on:

- AWS EC2
```