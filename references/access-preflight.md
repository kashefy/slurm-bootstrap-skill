# SSH and sudo preflight

From the administration machine, verify each node's SSH host key through a trusted source and add it to `known_hosts`. Configure `ProxyJump` in SSH config if the node is reached through a jump host.

If a jump host is required, confirm the SSH alias with the operator. Agree on an alias if it has yet to be configured, and have the operator verify the SSH config on the administration machine. Record the confirmed alias in the private lab notes or inventory, then reuse it in later SSH and Ansible commands. Use placeholders in shared examples.

If node addresses can change, such as with DHCP leases without guaranteed reservations, recheck recorded addresses when resuming work or after a stop/start. Compare each address with a current source that identifies the node independently of its old address, such as the provisioning platform, DHCP leases matched by MAC, or verified DNS. If an address changed, verify the node's SSH host key through a trusted source before connecting, update the affected SSH config, Ansible inventory, and peer name mappings, then verify peer name resolution from the nodes. For guaranteed static addresses or persistent DHCP reservations, skip this address-drift check and perform the usual SSH and peer name resolution checks.

After replacing `<admin_user>` and `<node>` with the intended account and node address or SSH alias, check noninteractive login with host-key verification. If passwordless sudo is the chosen escalation method, run both checks together.

```bash
ssh -o BatchMode=yes -o StrictHostKeyChecking=yes '<admin_user>@<node>' \
  'id -un && sudo -n id -u'
```

Expect the selected login account and UID `0` on separate lines. Resolve any access or privilege escalation failures before configuring Slurm.
