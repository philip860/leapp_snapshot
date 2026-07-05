# snapshot_aws

AWS EC2 EBS snapshot provider role for `philip860.leapp_snapshot`.

## Purpose

This role provides AWS EC2 EBS snapshot support before Red Hat Enterprise Linux
(RHEL) Leapp upgrade workflows.

The role performs AWS readiness checks, discovers the current EC2 instance,
identifies attached EBS volumes, and manages snapshots using the
`amazon.aws` Ansible collection.

This provider is normally invoked automatically by:

```yaml
philip860.leapp_snapshot.snapshot_dispatcher
```

when an AWS EC2 instance is detected.

---

# Supported actions

- check
- create
- remove
- revert

---

# Action behavior

## check

Validates AWS snapshot readiness.

The check action performs:

- AWS EC2 detection using Instance Metadata Service v2 (IMDSv2)
- AWS region discovery
- EC2 instance ID discovery
- Attached EBS volume discovery
- Requested volume validation

Example:

```yaml
snapshot_action: check
```

---

## create

Creates AWS EBS snapshots for EC2 attached volumes.

Example:

```yaml
snapshot_action: create
snapshot_set_name: leapp-preupgrade
```

By default, all attached EBS volumes are snapshotted.

Snapshots are automatically tagged:

```yaml
SnapshotSet: leapp-preupgrade
CreatedBy: philip860.leapp_snapshot
Purpose: RHEL Leapp upgrade
InstanceId: i-xxxxxxxx
VolumeId: vol-xxxxxxxx
```

---

## remove

Removes AWS EBS snapshots previously created by this collection.

Example:

```yaml
snapshot_action: remove
snapshot_set_name: leapp-preupgrade
```

Snapshots are discovered using:

```yaml
SnapshotSet
InstanceId
```

tags.

---

## revert

AWS revert support is intentionally protected.

Unlike LVM snapshots, AWS EBS rollback requires replacing attached volumes.

The restore workflow requires:

1. Stop EC2 instance
2. Create replacement volumes from snapshots
3. Detach current volumes
4. Attach restored volumes using original device names
5. Start EC2 instance

The current provider validates available restore snapshots but does not
automatically replace running volumes.

---

# Required AWS Authentication

## Ansible Automation Platform

Use the built-in AAP credential type:

```text
Amazon Web Services
```

Create the credential with the name:

```text
RHEL AWS Credential
```

Attach this credential to the snapshot Job Templates.

Example:

```yaml
credentials:
  - RHEL Machine Credential
  - RHEL AWS Credential
```

AAP automatically injects the standard AWS authentication variables:

```bash
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN
```

into the execution environment.

AWS credentials should not be stored in:

- playbooks
- inventory variables
- surveys
- extra_vars

Authentication should always be handled by the AAP credential system.

---

# Required AWS IAM Permissions

The AWS account requires permissions to manage EC2 snapshots.

Minimum IAM policy:

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

## Common snapshot variables

Normally provided by `snapshot_dispatcher`.

```yaml
snapshot_action: create

snapshot_set_name: leapp-preupgrade

snapshot_provider: auto
```

---

# AWS Provider Variables

## AWS region

Automatically detected from EC2 metadata.

Override if required:

```yaml
snapshot_aws_region: us-east-1
```

Default:

```yaml
snapshot_aws_region: ""
```

---

## EC2 instance ID

Automatically detected.

Override:

```yaml
snapshot_aws_instance_id: i-xxxxxxxx
```

Default:

```yaml
snapshot_aws_instance_id: ""
```

---

## Snapshot all attached EBS volumes

Default behavior:

```yaml
snapshot_aws_snapshot_all_volumes: true
```

The provider discovers all attached EBS volumes:

```text
EC2 Instance
     |
     +-- Root EBS volume
     |
     +-- Additional EBS volumes
```

---

# Specific volume selection

To snapshot only selected volumes:

```yaml
snapshot_aws_volume_ids:
  - vol-xxxxxxxx
  - vol-yyyyyyyy
```

If empty:

```yaml
snapshot_aws_volume_ids: []
```

all attached volumes are selected.

---

# Snapshot options

```yaml
snapshot_aws_description: "Leapp pre-upgrade snapshot"

snapshot_aws_wait: true

snapshot_aws_wait_timeout: 1800

snapshot_aws_delete_on_remove: true

snapshot_aws_fail_when_no_volumes: true
```

---

# Example Playbook Usage

Normally users should call the dispatcher.

```yaml
---
- name: Create RHEL upgrade snapshot
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
Create EBS Snapshot
```

---

# Example AAP Workflow

Recommended workflow:

```text
01 - Fleet Analysis
        |
        v
02 - Create Snapshot
        |
        v
03 - Leapp Upgrade
        |
        v
04 - Remove Snapshot
```

Snapshot job template:

```yaml
name: 02 - Create Snapshot

credentials:
  - RHEL Machine Credential
  - RHEL AWS Credential

extra_vars:
  snapshot_action: create
  snapshot_provider: auto
```

---

# Collection Requirements

Required collections:

```yaml
amazon.aws
```

Installed automatically through:

```yaml
dependencies:
  amazon.aws: "*"
```

from `galaxy.yml`.

---

# Supported Operating Systems

Guest operating systems:

- RHEL 7
- RHEL 8
- RHEL 9
- RHEL 10

Cloud provider:

- AWS EC2

Storage:

- Amazon EBS