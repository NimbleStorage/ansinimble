# NimbleStorage.Ansinimble
Manages software and products from Nimble Storage. This is BETA software.

## Requirements
This role assumes the latest Nimble Linux Toolkit (NLT) is installed on the host being managed by Ansible. The latest version of NLT may be obtained from [InfoSight](https://infosight.nimblestorage.com) (Nimble customers and partners only).

* Requires a Nimble iSCSI array with NimbleOS 3.3+
* Requires Ansible 2.2+ with the jmespath Python package installed

## Role Variables
The importance of variables depends on what objects are being managed.

These are the defaults that are broadly applicapable:
```
---
# These are required if NLT needs to be installed
nimble_group_options:
  ip-address: 192.168.59.64
  username: admin

# Please store password with Ansible Vault
nimble_group_password: admin

# Default Nimble folder options
nimble_folder_options_default:
  description: Folder created by Ansible

nimble_folder_update_options_default:
  description: "Folder updated by Ansible"

# These are the group facts gathered if no filters are applied
nimble_group_facts:
  access_control_records:
  arrays:
  folders:
  initiator_groups:
  initiators:
  performance_policies:
  pools:
  subnets:
  volumes:

# How to reach the Nimble group
nimble_group_http_scheme: https
nimble_group_http_port: 5392
nimble_group_api_version: v1
nimble_group_url: "{{ nimble_group_http_scheme }}://{{ nimble_host_facts.groups.main.group_ip }}:{{ nimble_group_http_port }}/{{ nimble_group_api_version }}"

# This is an optimistic guess, please provide discovery IP in complex setups
nimble_group_discovery_ip: "{{ nimble_group_facts | json_query('subnets[?allow_iscsi==`true`].discovery_ip | [0]') }}"

# Host locations
nimble_host_docker_conf_file: /opt/NimbleStorage/etc/docker-driver.conf
nimble_host_docker_driver_name: Docker-Driver
nimble_host_iqn_file: /etc/iscsi/initiatorname.iscsi
nimble_host_device_prefix: "/dev/nimblestorage"
nimble_linux_toolkit_prefix: /opt/NimbleStorage
nimble_linux_toolkit_options_default:
  - "silent-mode"
  - "accept-eula"
  - "ncm"

# Default initiator options
nimble_initiator_group_options_default:
  name: "{{ inventory_hostname_short }}"
  description: Created by Ansible for {{ inventory_hostname_short }}
  access_protocol: "{{ nimble_host_facts.groups.main.protocol }}"

nimble_initiator_options_default:
  label: "{{ inventory_hostname_short }}:sw-iscsi"
  access_protocol: "{{ nimble_host_facts.groups.main.protocol }}"
  iqn: "{{ nimble_host_facts.iqn }}"
  ip_address: "*"

# Default volume options
nimble_volume_mount_options: "defaults"
nimble_volume_mkfs_options: ""
nimble_volume_size: 1000
nimble_volume_options_default:
  description: "Volume created by Ansible"

# NimbleOS HTTP/JSON tunables
nimble_uri_retries: 6
nimble_uri_delay: 10
nimble_silence_payload: True
```

Each nimble_object and nimble_operation combo has a set of parameters that control what resources are being managed.

### nimble_object nimble_operation
Example usage of folder create:
```
- hosts: myhost
  roles:
    - { role: NimbleStorage.Ansinimble, nimble_object: folder, nimble_operation: create, nimble_folder: myfolder }
```

### folder create
Creates a Nimble folder.
```
nimble_folder: Folder name
nimble_folder_options: Dictionary of folder options
```

### folder update
Updates a Nimble folder.
```
nimble_folder: Folder name
nimble_folder_options: Dictionary of folder options to update
```

### folder delete
Deletes a Nimble folder.
```
nimble_folder: Folder name
```

### volume create
Create a new volume or clone from a snapshot.
```
nimble_volume: A 12 character volume name (XFS label limitation)
nimble_volume_from: Create volume from this source volume (name) 
nimble_volume_snapshot: Use this source snapshot to clone from (uses last snapshot if not specified)
nimble_volume_options: A dictionary of Nimble volume options (please see the REST documentation)
```

Note that the parameters `perfpolicy_id`, `pool_id` and `folder_id` accepts the vanity name as key in `nimble_volume_options`.

### volume map
Maps a volume to a host on the group.
```
nimble_volume: Volume to map
nimble_initiator_group_options: Options to apply to the initiator group (uses the node's IQN by default, hence this option is opional)
```

### volume mount
Mounts a volume on the host, creates a new filesystem if there is no filesystem on the volume.
```
nimble_volume: Nimble volume to mount
nimble_volume_mountpoint: Where to mount it
nimble_volume_mount_options: Mount options to put in /etc/fstab (optional)
nimble_volume_mkfs_options: Flags to pass to mkfs.xfs (optional)
```

### volume resize
Resizes (expands) a volume and filesystem.
```
nimble_volume: Volume name to resize
nimble_volume_size: Size in MiB to set volume to (currently no sanity checks)
nimble_volume_mountpoint: Where the volume is currently mounted
```

### volume update (not implemented yet)
Updates a volume.
```
nimble_volume: Volume name to change
nimble_volume_options: A dictionary of keys and values to update the volume with
```

### volume snapshot
Snapshots a Nimble volume.
```
nimble_volume: Volume to snapshot
nimble_volume_snapshot: Snapshot name (optional, uses timestamp)
```

### volume umount
Unmounts a filesystem on the host and disconnects the volume.
```
nimble_volume: Volume to disconnect
nimble_volume_mountpoint: Filesystem to umount
```

### volume unmap
Unmaps and offlines a volume on the Nimble group from an initiator.
```
nimble_volume: Volume to unmap and offline
```

### volume restore
Restore a Nimble volume from a snapshot. Requires volume to be umounted and unmapped.
```
nimble_volume: Volume to restore
nimble_volume_snapshot: Snapshot name to restore to (optional, uses last snapshot if not specified)
```

### volume delete
Permanently deletes a volume. Please umount and unmap first.
```
nimble_volume: Volume name to delete 
```

### host facts
Gathers facts about the host in `nimble_host_facts`.

Example:
```
ok: [smokey] => {
    "nimble_host_facts": {
        "docker": {
            "options": {
                "docker.volume.dir": "/opt/nimble/", 
                "nos.clone.snapshot.prefix": "BaseFor", 
                "nos.default.perfpolicy.name": "DockerDefault", 
                "nos.volume.destroy.when.removed": "false", 
                "nos.volume.name.prefix": "", 
                "nos.volume.name.suffix": ".docker"
            }, 
            "status": "DISABLED"
        }, 
        "groups": {
            "main": {
                "group_ip": "192.168.59.64", 
                "protocol": "iscsi", 
                "status": "enabled", 
                "user": "admin"
            }
        }, 
        "iqn": "iqn.1994-05.com.redhat:44be4d73789"
    }
}
```

**Note:** Requires NLT to be running, will attempt to start or install it if not.

### group facts
Gathers facts about the Nimble group suitable to perform `json_query` on.

Example playbook:
```
---
- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, nimble_object: group, nimble_operation: facts, nimble_group_facts: { pools: 'fields=all_flash'} }
  tasks:
    - debug: var=nimble_group_facts
```
Example output:
```
ok: [smokey] => {
    "nimble_group_facts": {
        "pools": [
            {
                "all_flash": false
            }
        ]
    }
}
```

Please see variables section for more info.

**Note:** By default `nimble_group_facts` gobbles the entire group. At scale this could become an issue. Always try to be smart about what you need from the group and only query for data that is needed for processing.

### nlt manage
Manages NLT on a host. 
```
nimble_linux_toolkit_state: Desired state, absent, present, running or stopped
nimble_linux_toolkit_bundle: Full path on control node to NLT installer (only required when present or running when not installed)
nimble_linux_toolkit_options: Additional installer flags (I.e docker or oracle)
```

## Dependencies
This role depends on the Nimble Linux Toolkit, please use a supported host OS by Nimble Linux Toolkit.

## Example Playbooks
Here are some example uses of this role. These are cut and paste from [examples](examples)

### sample_install.yml
Installs NLT.
```
---

# Provide nimble_linux_toolkit_bundle as an extra var to ansible-playbook, i.e:
# $ ansible-playbook -l limit -e nimble_linux_toolkit_bundle=/tmp/nlt_installer-2.0-0 sample_install.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, nimble_object: nlt, nimble_operation: manage, nimble_linux_toolkit_state: running }
```

### sample_uninstall.yml
Uninstalls NLT.
```
---
- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, nimble_object: nlt, nimble_operation: manage, nimble_linux_toolkit_state: absent }
```


### sample_debug.yml
Croaks out all host and group facts
```
---
- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, nimble_object: group, nimble_operation: facts }
  tasks:
    - debug: var=nimble_group_facts
    - debug: var=nimble_host_facts
```

### sample_group_fact.yml
Lightweight debugging.
```
---
- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, nimble_object: group, nimble_operation: facts, nimble_group_facts: { volumes: 'fields=name,serial_number' } }
  tasks:
    - debug: var=nimble_group_facts
```

### sample_provision.yml
Provisions and mounts a new volume.
```
---

# Provide nimble_volume and nimble_volume_mountpoint as extra vars, i.e:
# $ ansible-playbook -l limit -e nimble_volume=myvol1 -e nimble_volume_mountpoint=/mnt/myvol1 sample_provision.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, nimble_object: volume, nimble_operation: create }
    - { role: NimbleStorage.Ansinimble, nimble_object: volume, nimble_operation: map }
    - { role: NimbleStorage.Ansinimble, nimble_object: volume, nimble_operation: mount }
```

### sample_snapshot.yml
```
---

# Provide nimble_volume and nimble_volume_snapshot as extra vars, i.e:
# $ ansible-playbook -l limit -e nimble_volume=myvol1 -e nimble_volume_snapshot=mysnap1 sample_snapshot.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, nimble_object: volume, nimble_operation: snapshot }
```
### sample_restore.yml
```
---

# Provide nimble_volume, nimble_volume_mountpoint and nimble_volume_snapshot as extra vars, i.e
# $ ansible-playbook -l limit -e nimble_volume=myvol1 -e nimble_volume_mountpoint=/mnt/myvol1 -e nimble_volume_snapshot=mysnap1 sample_restore.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, nimble_object: volume, nimble_operation: umount }
    - { role: NimbleStorage.Ansinimble, nimble_object: volume, nimble_operation: unmap }
    - { role: NimbleStorage.Ansinimble, nimble_object: volume, nimble_operation: restore }
    - { role: NimbleStorage.Ansinimble, nimble_object: volume, nimble_operation: map }
    - { role: NimbleStorage.Ansinimble, nimble_object: volume, nimble_operation: mount }
```


### sample_resize.yml
```
---

# Provide nimble_volume and nimble_volume_size as extra vars, i.e:
# $ ansible-playbook -e limit -e nimble_volume=myvol1 -e nimble_volume_size=2000 sample_resize.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, nimble_object: volume, nimble_operation: resize }
```

### sample_decommision.yml
Unmounts and deletes a volume.
```
---

# Provide nimble_volume and nimble_volume_mountpoint as extra vars, i.e:
# $ ansible-playbook -l limit -e nimble_volume=myvol1 -e nimble_volume_mountpoint=/mnt/myvol1 sample_decommission.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, nimble_object: volume, nimble_operation: umount }
    - { role: NimbleStorage.Ansinimble, nimble_object: volume, nimble_operation: unmap }
    - { role: NimbleStorage.Ansinimble, nimble_object: volume, nimble_operation: delete }
```

## Security considerations
Always secure your `nimble_group_password` with Ansible Vault to prevent eavesdropping. 

## Contributing
Please feel free to submit pull requests. Include a test in [tests/test.yml](tests/test.yml) if including new functionality. Tests are run manually before merge.
 
## License
Apache 2.0, please see [LICENSE](LICENSE)

## Version
Please see [VERSION](VERSION)

## Author Information
This role is maintained by Michael Mattsson, a Nimble Storage employee, on his spare time. Please use the GitHub issue tracker for any queries related to this role.
