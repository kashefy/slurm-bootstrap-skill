# Slurm version policy

- When choosing Ubuntu for a fresh cluster, evaluate the latest released LTS and check which Slurm maintenance release its APT repositories provide. Check the upstream support window and choose the distribution packages if they meet the lab's needs. If the operator wants a newer supported Slurm release, build Debian packages from the pinned SchedMD source for the selected Ubuntu release.

- Before installation, record the target Slurm maintenance release and package source. Check package candidates on every host. For a fresh cluster, use the same maintenance release for the controller, compute daemons, accounting daemon if enabled, and client commands. Install plugins built for that release.
- Verify installed versions on each host before validating scheduling and accounting.
- On fresh Debian or Ubuntu installations, suppress package-triggered service starts until configuration is ready. Use `ansible.builtin.apt` with `policy_rc_d: 101`, or apply an equivalent temporary Debian service-start policy around a direct `apt-get install` while preserving any existing policy. Start and validate the chosen authentication service, deploy `slurm.conf` or set up configless discovery, start the controller, then start the compute daemons. Without a local configuration or a working configless path, premature `slurmd` startup can retry DNS SRV discovery until systemd times out. [Ansible APT service policy](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/apt_module.html#parameter-policy_rc_d)
- Before choosing the configuration path, inspect the installed units and drop-ins with `systemctl cat`. Check each referenced optional environment file only if it exists. Treat a missing optional file or a search with zero matches as normal, and report errors reading existing files. For cgroup v2 CPU and memory limits, confirm the compute nodes expose `cpuset` and `memory` in `/sys/fs/cgroup/cgroup.controllers`.
- For upgrades or mixed versions, verify compatibility and upgrade order for the releases involved. Back up controller state, Slurm configuration, and the accounting database if enabled. Upgrade the Slurm packages on each host together. [SchedMD upgrade guide](https://slurm.schedmd.com/upgrades.html)

## Building Debian packages from SchedMD source

1. Check for `bzip2` on the build host and select it explicitly with `tar -I bzip2` for a `.tar.bz2` release archive. This avoids relying on the host's default decompressor.
2. If passing `Build-Depends` to `apt-get satisfy`, normalize whitespace and remove its trailing comma before appending more dependencies. Preserve alternative dependencies and version constraints. Simulate the transaction first and run `dpkg-checkbuilddeps` after installing dependencies.
3. Review the build log and Lintian findings. Verify the generated packages' names, versions, and architectures. Investigate errors and warnings relevant to the selected packages, and validate any affected plugins the deployment uses.
4. Stage local packages in a directory that `_apt` can traverse and read, with the files protected from unprivileged writes. Simulate installation of the role-specific package set and review the transaction before applying it.

### Archive a validated package build

Advise the operator to archive locally built packages after checking their metadata and before cleaning up the build or staging directories. Keep the exact packages in backed-up storage so new nodes can join at the cluster's deployed version and existing roles can be restored without rebuilding. Include the packages used by each enabled role, the build log, the source URL, and the target distribution and architecture.

Give the operator commands adapted to their actual artifact paths and role packages. Run the following template in Bash where the verified build artifacts are available, with `build_dir`, `archive_dir`, `build_log`, `source_url`, `package_version`, and `target_platform` set to the actual values. Set the Bash array `package_files` to the exact `.deb` basenames. Use a new, version-specific archive directory with an existing parent.

```bash
(
  set -e
  mkdir -m 0700 -- "$archive_dir"
  cd "$build_dir"
  cp -- "${package_files[@]}" "$build_log" "$archive_dir/"
  cd "$archive_dir"
  printf 'Source: %s\nPackage version: %s\nTarget: %s\n' \
    "$source_url" "$package_version" "$target_platform" > BUILD-INFO.txt
  sha256sum -- "${package_files[@]}" > SHA256SUMS
  sha256sum -c SHA256SUMS
)
```

Use an absolute archive path. Before deploying from the archive, run `(cd "$archive_dir" && sha256sum -c SHA256SUMS)`, confirm the target OS and architecture, then stage and simulate installation of the packages for that role.
