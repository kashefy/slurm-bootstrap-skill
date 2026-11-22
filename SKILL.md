---
name: slurm-bootstrap
description: Guide Slurm installation, configuration, and validation on existing SSH-reachable Linux nodes.
---

# slurm-bootstrap

**Status: Draft**

Start with existing SSH-reachable Linux nodes and an administration machine. Use placeholders such as `<cluster_name>` for site-specific values.

If the operator is still choosing Ubuntu for a fresh lab, evaluate the latest released LTS and its Slurm package candidate using the [version policy](references/version-policy.md).

## Workflow outline

1. Confirm node roles, [SSH access and privilege escalation](references/access-preflight.md), stable name resolution, synchronized clocks, and consistent user and group IDs.
2. Apply the [OS update and reboot guidance](references/node-maintenance.md#before-installing-slurm). Select the Slurm release and package source, then apply the [version rules](references/version-policy.md) before installing Slurm packages.
3. Configure authentication, the controller, compute nodes, client access, and accounting if enabled. Use Ansible for repeatable multi-host checks and configuration where available.
4. Verify daemon and client versions, inspect node and partition state with `sinfo`, and submit a small job. For a multi-node cluster, verify a job runs across nodes. If accounting is enabled, confirm the completed job appears in `sacct`.

The outline describes the intended workflow. Lab-verified procedures will be added as they become available.

For labs that simulate resource-based scheduling, choose compute nodes with different vCPU and RAM allocations when planning the lab. Before configuring Slurm, verify the CPU topology and usable memory inside each node. In production, derive each node's configuration from its installed hardware and any limits enforced by its runtime environment. For both environments, keep configured schedulable memory within the detected capacity and allow headroom for the OS. In a heterogeneous lab, test CPU and memory placement separately with single-node requests that fit only the larger node.

For nodes already serving workloads, follow the [rolling maintenance guidance](references/node-maintenance.md#maintaining-an-active-cluster) before applying updates that can restart services or require a reboot.

The preflight and post-deployment validation structure draws on [NVIDIA DeepOps](https://github.com/NVIDIA/deepops/blob/master/skills/deploy-airgapped/SKILL.md).
