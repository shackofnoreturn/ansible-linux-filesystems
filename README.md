# Ansible Linux Filesystems

*Reusable Ansible playbooks and roles for configuring NFS servers and clients on Debian/Ubuntu Linux hosts.*


## What It Does
### playbook-nfs-server.yml
- Installs the NFS server package
- Refuses to configure exports whose paths are not mounted
- Writes `/etc/exports`
- Installs a systemd timer:
    - checks storage availability
    - refreshes exports

### playbook-nfs-client.yml
- Installs the NFS client package
- Creates mount-point directories
- Writes persistent NFS mount entries to `/etc/fstab` using `ansible.posix.mount`

The NFS client role uses `state: present`, so it configures persistent mounts but does not itself mount them immediately during the play.

### playbook-samba-client.yml / playbook-samba-server.yml
Samba playbooks and roles are present as placeholders but currently have no tasks and do not configure Samba.

### mount
The generic `mount` role is also currently empty.


## Compatibility with shackhq-ansible-infrastructure

The NFS playbooks are structurally compatible with the infrastructure inventories:

| Purpose | Inventory | Group passed as `target_host` |
| --- | --- | --- |
| NFS servers | `inventories/infrastructure/inventory.yml` | `nfs_servers` |
| NFS clients | `inventories/management/inventory.yml` | `nfs_clients` |

The playbooks require `target_host`; set it in the Semaphore template's extra variables. Inventory host vars for `shack-fil-p001` and `shack-fil-p002` already use the `nfs_exports` data structure consumed by the server role.


## Semaphore

Configure Semaphore so the task template checks out this repository as its **playbook repository** and uses the appropriate file inventory from `shackhq-ansible-infrastructure`.

Create separate task templates to keep server and client targeting explicit:

| Setting | NFS server template | NFS client template |
| --- | --- | --- |
| Playbook repository | `ansible-linux-filesystems` | `ansible-linux-filesystems` |
| Playbook | `playbook-nfs-server.yml` | `playbook-nfs-client.yml` |
| File inventory | `inventories/infrastructure/inventory.yml` | `inventories/management/inventory.yml` |
| Extra variable | `target_host: nfs_servers` | `target_host: nfs_clients` |


## Roles And Variables

| Role | Main variables | Purpose |
| --- | --- | --- |
| `nfs-server` | `nfs_exports`, `nfs_storage_mounts`, `nfs_storage_recovery_interval` | Validate storage, configure exports, and install storage recovery units. |
| `nfs-client` | `nfs_mounts`, `nfs_mount_defaults` | Install client tools and configure persistent mounts. |

The server export entries contain a `path` and a `clients` list; each client item has `network` and `options`. Client mount entries contain `server`, `export`, and `mountpoint`, with optional per-mount `fstype`, `options`, `owner`, `group`, and `mode` values.
