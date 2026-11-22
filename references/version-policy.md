# Slurm version policy

- Before installation, record the target Slurm maintenance release and package source. Check package candidates on every host. For a fresh cluster, use the same maintenance release for the controller, compute daemons, accounting daemon if enabled, and client commands. Install plugins built for that release.
- Verify installed versions on each host before validating scheduling and accounting.
- For upgrades or mixed versions, verify compatibility and upgrade order for the releases involved. Back up controller state, Slurm configuration, and the accounting database if enabled. Upgrade the Slurm packages on each host together. [SchedMD upgrade guide](https://slurm.schedmd.com/upgrades.html)
