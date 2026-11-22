# Slurm version policy

- When choosing Ubuntu for a fresh cluster, evaluate the latest released LTS and check which Slurm maintenance release its APT repositories provide. Check the upstream support window and choose the distribution packages if they meet the lab's needs. If the operator wants a newer supported Slurm release, build Debian packages from the pinned SchedMD source for the selected Ubuntu release.

- Before installation, record the target Slurm maintenance release and package source. Check package candidates on every host. For a fresh cluster, use the same maintenance release for the controller, compute daemons, accounting daemon if enabled, and client commands. Install plugins built for that release.
- Verify installed versions on each host before validating scheduling and accounting.
- For upgrades or mixed versions, verify compatibility and upgrade order for the releases involved. Back up controller state, Slurm configuration, and the accounting database if enabled. Upgrade the Slurm packages on each host together. [SchedMD upgrade guide](https://slurm.schedmd.com/upgrades.html)

## Building Debian packages from SchedMD source

1. Check for `bzip2` on the build host and select it explicitly with `tar -I bzip2` for a `.tar.bz2` release archive. This avoids relying on the host's default decompressor.
2. If passing `Build-Depends` to `apt-get satisfy`, normalize whitespace and remove its trailing comma before appending more dependencies. Preserve alternative dependencies and version constraints. Simulate the transaction first and run `dpkg-checkbuilddeps` after installing dependencies.
3. Review the build log and Lintian findings. Verify the generated packages' names, versions, and architectures. Investigate errors and warnings relevant to the selected packages, and validate any affected plugins the deployment uses.
4. Stage local packages in a directory that `_apt` can traverse and read, with the files protected from unprivileged writes. Simulate installation of the role-specific package set and review the transaction before applying it.
