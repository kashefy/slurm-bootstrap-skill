# OS updates and reboots on Slurm nodes

## Before installing Slurm

- After SSH and privilege escalation checks pass, update the target nodes using the distribution's package manager. Use the same approved repository baseline across comparable nodes. Use a shared repository snapshot or package pins when exact versions matter. If nodes came from a VM image, refresh the image on its own schedule and check each guest's package and reboot state after first boot.
- Prefer a small Ansible prep playbook for repeated updates. Scope it to the intended hosts, refresh package metadata, apply the planned updates, and check the distribution's reboot indicator after updating. On Ubuntu, check `/run/reboot-required` even if the package task made no changes, since first-boot updates may already have set it. Use `ansible.builtin.reboot` when needed to verify the guest restarts and responds again. Check the running kernel after a kernel update. [Ansible reboot module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/reboot_module.html)

## Maintaining an active cluster

- Review the planned package changes for service restarts and Slurm version changes. Handle Slurm version changes under the [version policy](version-policy.md). Plan controller and accounting service maintenance around their availability and failover arrangements.
- Patch compute nodes in batches sized to preserve the capacity required by the workload. Drain each batch before applying updates that may interrupt jobs, and wait for existing jobs to finish and nodes to reach `DRAINED`. Track which nodes were drained for this maintenance. [SchedMD node state reference](https://slurm.schedmd.com/scontrol.html)
- After patching and any required reboot, verify connectivity and `slurmd` health. Return healthy nodes drained for this maintenance to service, confirm they register and become schedulable, and run a targeted smoke job before advancing. Keep failed nodes drained for diagnosis. [SchedMD node state reference](https://slurm.schedmd.com/scontrol.html)
