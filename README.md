# NimbleStorage.Ansinimble
Manages software and products from Nimble Storage. This is BETA software.

## Requirements
Ansinimble supports two different product lines. HPE Nimble Storage arrays and HPE Cloud Volumes. Requirements vary slightly between products. 
### Nimble Arrays
This role assumes the latest Nimble Linux Toolkit (NLT) is installed on the host being managed by Ansible. The latest version of NLT may be obtained from [InfoSight](https://infosight.nimblestorage.com) (Nimble customers and partners only).

* Requires a Nimble iSCSI array with NimbleOS 3.3+
* Requires Ansible 2.2.3+ with the jmespath Python package installed (please see deprecation notes on Ansible 2.5 below)

### Cloud Volumes
Host requirements are less stringent for Cloud Volumes. The Linux Connection Manager found on the Cloud Volumes portal needs to be installed but there is a task that can do that for you.

* Requires API access to HPE Cloud Volumes
* Requires Ansinimble 2.2.3+ with the jmespath and boto (for EC2) Python package installed on the Ansible host. (please see deprectation notes on Ansible 2.5 below)

## Deprecation notes and tested versions of Ansible
The following versions of Ansible are currently being tested:
* 2.2.3.0
* 2.3.3.0
* 2.4.5.0
* 2.5.5

When Ansible 2.9 becomes generally available, the role will be re-written and only support Ansible 2.5+. A number of deprecation warnings will be present when executing under Ansible 2.4 and 2.5, they are harmless. Deprecation warnings may safely be disabled.

**Important:** When executing under Ansible 2.5, `ansible-playbook -b` (become) must be used for the role to work properly. This is due to a change in the `include` statement.

# Reference Documentation Tables of Contents 

* [Role Variables](#role-variables)
* [Usage](#usage)
 * [Nimble Storage Array Operations](#nimble-storage-array-operations)
    * [nimble_object nimble_operation](#nimble_object-nimble_operation)
    * [folder create](#folder-create)
    * [folder update](#folder-update)
    * [folder delete](#folder-delete)
    * [volume create](#volume-create)
    * [volume map](#volume-map)
    * [volume mount](#volume-mount)
    * [volume resize](#volume-resize)
    * [volume update (not implemented yet)](#volume-update-not-implemented-yet)
    * [volume snapshot](#volume-snapshot)
    * [volume umount](#volume-umount)
    * [volume unmap](#volume-unmap)
    * [volume restore](#volume-restore)
    * [volume delete](#volume-delete)
    * [volume protect](#volume-protect)
    * [volume unprotect](#volume-unprotect)
    * [volcoll create](#volcoll-create)
    * [volcoll prune](#volcoll-prune)
    * [prottmpl create](#prottmpl-create)
    * [prottmpl delete](#prottmpl-delete)
    * [replication partner](#replication-partner)
    * [replication divorce](#replication-divorce)
    * [cloud partner](#cloud-partner)
    * [cloud divorce](#cloud-divorce)
    * [host facts](#host-facts)
    * [group facts](#group-facts)
    * [nlt manage](#nlt-manage)
 * [Cloud Volume Operations](#cloud-volume-operations)
    * [cloud_object cloud_operation](#cloud_object-cloud_operation)
    * [volume create](#volume-create-1)
    * [volume update](#volume-update)
    * [volume map](#volume-map-1)
    * [volume mount](#volume-mount-1)
    * [volume resize](#volume-resize-1)
    * [volume snapshot](#volume-snapshot-1)
    * [volume umount](#volume-umount-1)
    * [volume unmap](#volume-unmap-1)
    * [volume restore](#volume-restore-1)
    * [volume delete](#volume-delete-1)
    * [portal facts](#portal-facts)
    * [lcm manage](#lcm-manage)
    * [array setup](#array-setup)
    * [password update](#password-update)
    * [host facts](#host-facts-1)
 * [Example Playbooks](#example-playbooks)
 * [Nimble Storage Array Group Examples](#nimble-storage-array-group-examples)
    * [nimble/sample_install.yml](#nimblesample_installyml)
    * [nimble/sample_uninstall.yml](#nimblesample_uninstallyml)
    * [nimble/sample_debug.yml](#nimblesample_debugyml)
    * [nimble/sample_group_fact.yml](#nimblesample_group_factyml)
    * [nimble/sample_provision.yml](#nimblesample_provisionyml)
    * [nimble/sample_snapshot.yml](#nimblesample_snapshotyml)
    * [nimble/sample_restore.yml](#nimblesample_restoreyml)
    * [nimble/sample_resize.yml](#nimblesample_resizeyml)
    * [nimble/sample_decommision.yml](#nimblesample_decommisionyml)
    * [nimble/sample_array_setup.yml](#nimblesample_array_setupyml)
    * [nimble/sample_password_change.yml](#nimblesample_password_changeyml)
    * [nimble/sample_protect.yml](#nimblesample_protectyml)
    * [nimble/sample_unprotect.yml](#nimblesample_unprotectyml)
    * [nimble/sample_volcoll_create.ym](#nimblesample_volcoll_createym)
    * [nimble/sample_volcoll_prune.yml](#nimblesample_volcoll_pruneyml)
    * [nimble/sample_prottmpl_create.yml](#nimblesample_prottmpl_createyml)
    * [nimble/sample_prottmpl_delete.yml](#nimblesample_prottmpl_deleteyml)
    * [nimble/sample_nimble_partner.yml](#nimblesample_nimble_partneryml)
    * [nimble/sample_nimble_divorce.yml](#nimblesample_nimble_divorceyml)
    * [nimble/sample_cloud_partner.yml](#nimblesample_cloud_partneryml)
    * [nimble/sample_cloud_divorce.yml](#nimblesample_cloud_divorceyml)
    * [nimble/sample_store_create.yml](#nimblesample_store_createyml)
    * [nimble/sample_store_delete.yml](#nimblesample_store_deleteyml)
    * [nimble/sample_store_resize.yml](#nimblesample_store_resizeyml)
 * [Cloud Volume Examples](#cloud-volume-examples)
    * [cloud/sample_install.yml](#cloudsample_installyml)
    * [cloud/sample_provision.yml](#cloudsample_provisionyml)
    * [cloud/sample_resize.yml](#cloudsample_resizeyml)
    * [cloud/sample_update.yml](#cloudsample_updateyml)
    * [cloud/sample_snapshot.yml](#cloudsample_snapshotyml)
    * [cloud/sample_restore.yml](#cloudsample_restoreyml)
    * [cloud/sample_decommission.yml](#cloudsample_decommissionyml)
    * [cloud/sample_uninstall.yml](#cloudsample_uninstallyml)
    * [cloud/sample_debug.yml](#cloudsample_debugyml)

## Role Variables
The importance of variables depends on what objects are being managed.

These are the defaults that are broadly applicapable or serving as examples:
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
  description: "Folder created by Ansible"

nimble_folder_update_options_default:
  description: "Folder updated by Ansible"

# Default volcoll options (when created standalone)
nimble_volcoll_options_default:
  description: "Volume Collection created by Ansible"

# Default protection template options
nimble_prottmpl_options_default:
  description: "Protection Template created by Ansible"

nimble_protsched_options_default:
  description: "Protection Schedule created by Ansible"
  volcoll_or_prottmpl_type: protection_template

# Default partnership options as fallbacks
nimble_partnership_options_default:
  upstream:
    description: "Established by Ansible"
    subnet_label: mgmt
  downstream:
    description: "Established by Ansible"
    subnet_label: mgmt
    name: upstream
    hostname: "{{ nimble_group_options['ip-address'] }}"
    username: "{{ nimble_group_options.username }}"
    password: "{{ nimble_group_password }}"

# Default cloud partnership options
nimble_cloud_partnership_options_default:
  description: "Established by Ansible"
  subnet_label: mgmt-data
  partner_type: tunnel_endpoint

# These are the group facts gathered if no filters are applied with nimble_group_facts
nimble_group_facts_default:
  access_control_records:
  arrays:
  folders:
  initiator_groups:
  initiators:
  performance_policies:
  pools:
  subnets:
  volumes:
  volume_collections:
  protection_templates:

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
nimble_host_device_prefix: /dev/nimblestorage
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
nimble_volume_mount_options: "_netdev,auto"
nimble_volume_mkfs_options: ""
nimble_volume_size: 1000
nimble_volume_options_default:
  description: "Volume created by Ansible"

# Device retries
nimble_device_retries: 3
nimble_device_delay: 10

# NimbleOS HTTP/JSON tunables
nimble_uri_retries: 6
nimble_uri_delay: 10
nimble_silence_payload: True

# NimbleOS setup from scratch
nimble_array_config:
  XX-000000:                                 # This is the serial number of the new array
    group_name: group-sjc-tme000-va          # Name of the group.
    name: sjc-tme000-va                      # Name of the array.
    domainname: vlab.nimblestorage.com       # DNS domain name of this system.
    ntpserver: time.nimblestorage.com        # Hostname or IP address of NTP server.
    timezone: America/Los_Angeles            # Time zone that the array is in.
    mgmt_ipaddr: 10.18.173.174               # Management IP address.
    default_gateway: 10.18.160.1             # Default gateway IP address.

    dnsservers:                              # IP addresses of the DNS servers.
      - 10.18.5.4
      - 10.12.255.254

    iscsi_automatic_connection_method: "yes" # Redirect connections from the specified
                                             # iSCSI target IP appropriately.

    iscsi_connection_rebalancing: "yes"      # Rebalance iSCSI connections by periodically
                                             # breaking existing connections that are
                                             # out-of-balance.

    subnets:                                 # Subnets to configure (In NIC order)
      - label: mgmt-data                     # Subnet Label on NIC.
        subnet_addr: 10.18.173.174/19        # Subnet on NIC.
        subnet_type: management              # Type of subnet -- data, management, or data
                                             # and management
        subnet_mtu: 1500                     # Size of MTU - standard, jumbo, other.

      - label: mgmt-data                     # Subnet Label on NIC.
        subnet_addr: 10.18.173.174/19        # Subnet on NIC.
        subnet_type: management              # Type of subnet -- data, management, or data
                                             # and management
        subnet_mtu: 1500                     # Size of MTU - standard, jumbo, other.

      - label: data1                         # Subnet Label on NIC.
        data_ipaddr: 172.16.237.135          # Data IP address setting.
        subnet_addr: 172.16.237.135/19       # Subnet on NIC.
        subnet_type: data                    # Type of subnet -- data, management, or data
                                             # and management
        subnet_mtu: 1500                     # Size of MTU - standard, jumbo, other.

      - label: data2                         # Subnet Label on NIC.
        data_ipaddr: 172.16.45.159           # Data IP address setting.
        subnet_addr: 172.16.45.159/19        # Subnet on NIC.
        subnet_type: data                    # Type of subnet -- data, management, or data
                                             # and management
        subnet_mtu: 1500                     # Size of MTU - standard, jumbo, other.

    discovery_ipaddrs:                       # Discovery IP address setting.
      - 172.16.237.136
      - 172.16.45.160

    support_ipaddrs:
      - 10.18.173.175                        # Support IP address setting node A.
      - 10.18.173.176                        # Support IP address setting node B.

# Cloud Volumes
cloud_portal_http_host: cloudvolumes.hpe.com

# Please protect your credentials with Ansible Vault
cloud_portal_access_key: nimble
cloud_portal_secret_key: nimblestorage

cloud_lcm_installer: https://ncv.cloud.nimblestorage.com/tools/ncv_installer_1.0.0.16
cloud_lcm_prefix: /opt/NimbleStorage
cloud_host_iqn_file: /etc/iscsi/initiatorname.iscsi

# Cloud Portal
cloud_portal_http_scheme: https
cloud_portal_http_port: 443
cloud_portal_api_version: v2
cloud_portal_url: "{{ cloud_portal_http_scheme }}://{{ cloud_portal_http_host }}:{{ cloud_portal_http_port }}"
cloud_portal_api_endpoint: "{{ cloud_portal_url }}/api/{{ cloud_portal_api_version }}"

# HTTP/JSON tunables
cloud_uri_retries: 24 
cloud_uri_delay: 10
cloud_silence_payload: True

# Default Cloud Portal facts
cloud_portal_facts_default:
  cloud_volumes:
  regions:
  session:
  initiators:
  interfaces:
  onprem_replication_partners:
  replication_partnerships:
  replication_stores:
  providers:

# Default Cloud Volume options
cloud_volume_mount_options: "_netdev,auto"
cloud_volume_mkfs_options: ""
cloud_volume_size: 10000
cloud_volume_from_default: {}
cloud_volume_resize_body: {}

cloud_volume_options_default:
  volume_type: PF
  encryption: False
  schedule: none
  retention: 0
  perf_policy: "Windows File Server"
  iops: 1000

cloud_volume_mandatory_options:
  - existing_cloud_subnet
  - private_cloud 

cloud_replica_options_default:
  iops: 1000

cloud_replica_mandatory_options:
  - existing_cloud_subnet
  - private_cloud 

cloud_replica_snapshot_index: 0

# Default Replication Store options
cloud_store_options_default:
  volume_type: GPF
cloud_store_size: 1099776
cloud_store_size_min: 1099776
cloud_store_size_max: 32985088
cloud_store_region: us-test

# Cloud host defaults
cloud_host_device_prefix: /dev/nimblestorage
cloud_device_retries: 3
cloud_device_delay: 10
```

## Usage
Managing Nimble Storage arrays and Cloud Volumes vary slightly as consuming storage as a service vs infrastructure are two separate domains. The most significant difference is that the `object` and `operation` is prefixed either `nimble` or `cloud`. The role should be able to manage both if you have hosts connected to both on premises Nimble Storage arrays and Cloud Volumes.

## Nimble Storage Array Operations
Each `nimble_object` and `nimble_operation` combo has a set of parameters that control what resources are being managed.

### nimble_object nimble_operation
Example usage of folder create:
```
- hosts: myhost
  vars: 
    nimble_folder: myfolder
  roles:
    - { role: NimbleStorage.Ansinimble,
        nimble_object: folder, 
        nimble_operation: create 
      }
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
nimble_volume: An ASCII string (some special characters allowed)
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

### volume protect
Protects a volume in either a dedicated volume collection with a protection template or an existing volume collection.
```
nimble_volume: Volume name to protect
nimble_protection_template: Name of protection template to create a new volume collection with
nimble_volume_collection: Existing volume collection to add the volume to
```
**Note**: `nimble_protection_template` and `nimble_volume_collection` are mutually exclusive.

### volume unprotect
Remove the volume collection association from a volume.
```
nimble_volume: Volume name to remove volume collection association from
```

### volcoll create
Create a new volume collection from a protection template.
```
nimble_volume_collection: Name of the volume collection
nimble_volcoll_options: Key/value of volcoll options (see API documentation on HPE InfoSight)
```

### volcoll prune
Prunes the array group of orphaned volume collections. No parameters needed.

### prottmpl create
Creates a protection schedule with a set of schedules and replica destinations.
```
nimble_protection_template: Name of the protection template
nimble_protection_template_options: Key/value pairs of protection template options (see API documentation on HPE InfoSight)
nimble_protection_schedules:
  myschedule:
    downstream_partner_id: Name of downstream partner (optional)
    num_retain_replica: Number of snapshots to retain on the replica (optional)
    description: A human readable description of schedule (optional)
    period_unit: A human readable unit of periods such as "minutes", "hours", "days" or "weeks"
    days: A human readable comma separated list of days to take snapshots, example: "saturday,sunday"
    at_time: Non-negative integer in range [0,86399] which is equivalent to [0:00:00 AM, 23:59:59 PM]
    until_time: Time of day to stop taking snapshots (same range as above). Only applicable for units < "days".
    num_retain: Number of snapshots to retain in this schedule
  myotherschedule:
    ...
```
**Note**: More options are availble, please see the full API documentation on HPE InfoSight.

### prottmpl delete
Deletes a protection template.
```
nimble_protection_template: Name of protection template to delete
```

### replication partner
Partner with a downstream array group.
```
nimble_partner: Name of partnership to marry, example: mypartnership
nimble_partnerships:
  mypartnership:
    upstream:
      name: Vanity name of upstream group, example: group-sjc-array869
      hostname: FQDN of upstream array
      subnet_label: Label of network to use for replication
      description: "My upstream array"
    downstream:
      name: Vanity name of downstream group, example: group-sjc-array875
      hostname: FQDN of downstream array
      subnet_label: Label of network to use for replication
      username: Username on downstream array
      password: Password on downstream array
      description: "My downstream array"
    secret: Secret to use for pairing (minimum 8 chars)
```

### replication divorce
Divorce an array group (may not be referenced by any protection templates)
```
nimble_partner: Replication partner name to delete 
```
**Note:** Does not use nimble_partnerships data structure or communicates to both ends of the partnership

### cloud partner
Partner an array group with Cloud Volumes.
```
cloud_store: Name of "Replication Store" in cloud volumes to use (will be created with default options if it doesn't exist)
cloud_store_region: Cloud Volumes region to use
```
**Note**: Require Cloud Volumes authentication variables to be set too

### cloud divorce
Divorces a partnership with Cloud Volumes (may not have any protection templates associated).
```
cloud_store: Name of cloud store to divorce from
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
**Note:** Calling group facts with nimble_group_facts multiple times in the same play will have undesired effects. If a certain fact(s) need to be refreshed mid-play, use nimble_group_fact_refresh with the keys needed to be refreshed.
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

## Cloud Volume Operations
Each `cloud_object` and `cloud_operation` combo has a set of parameters that control what resources are being managed.

### cloud_object cloud_operation
Example usage of managing LCM (Linux Connection Manager)
```
- hosts: myhost
  roles:
    - { role: NimbleStorage.Ansinimble, cloud_object: lcm, cloud_operation: manage, cloud_lcm_state: present }
```

### volume create
Creates or clones a Cloud Volume. This operation assumes a couple of default `cloud_volume_options` from the defaults definition. Please see the Cloud Volumes API documentation for full list of options.
```
cloud_volume: Cloud Volume to create
cloud_volume_from: Create Cloud Volume from this source volume (name) 
cloud_volume_snapshot: Use this source snapshot to clone from (uses last snapshot if not specified)
cloud_volume_provider: Amazon AWS or Microsoft Azure
cloud_volume_region: Which region to expose your volume, i.e us-west-1
cloud_volume_options: Below are the mandatory keys
  private_cloud: VPC or VNET
  existing_cloud_subnet: Subnet CIDR notation 
```
### volume update
Updates a Cloud Volume, such as changing the IOPS limit.
```
cloud_volume: Cloud Volume to update
cloud_volume_options: Key and value to update. Full list of options available in the Cloud Volumes API documentation.
  key: value
```

### volume map
Maps a Cloud Volume to a cloud instance. (Does not login to the target)
```
cloud_volume: Cloud Volume to map
cloud_host_ip: IP address of your cloud instance
```

### volume mount
Mounts a Cloud Volume on a cloud instance. Will create a new filesystem (XFS) and add an entry to fstab. 
```
cloud_volume: Cloud Volume to mount (needs to be mapped prior)
cloud_volume_mountpoint: Where to mount the filesystem
```

### volume resize
Resizes a Cloud Volume and resizes the filesystem. Needs to be mounted to succeed.
```
cloud_volume: Cloud Volume to resize
cloud_volume_mountpoint: Where the Cloud Volume is mounted 
cloud_volume_size: Size in total MB to set the volume to (only expansion)
```

### volume snapshot
Snapshots a Cloud Volume
```
cloud_volume: Cloud Volume to snapshot
cloud_volume_snapshot: Optional snapshot name (uses a timestamp by default)
```

### volume umount
Unmounts a Cloud Volume from a cloud instance and disconnects the volume from the host and wipes the fstab entry.
```
cloud_volume: Cloud Volume to unmount
cloud_volume_mountpoint: Where the Cloud Volume will be umounted from
```

### volume unmap
Unmaps a Cloud Volume from a cloud instance.
```
cloud_volume: Cloud Volume to unmap
cloud_host_ip: The cloud instance IP address to unmap
```

### volume restore
Restores a Cloud Volume from a known snapshot or last snapshot. The cloud volume needs to be unmounted and unmapped from the host prior.
```
cloud_volume: Cloud Volume to restore
cloud_volume_snapshot: Optional snapshot name, will default to last snapshot on the volume if not specified
```

### volume delete
Deletes a Cloud Volume. Needs to be unmounted and unmapped from the host prior.
```
cloud_volume: Cloud Volume to delete
```

### portal facts
Scrapes the facts off the Cloud Volume Portal for use with `json_query` filters. Primarily an internal task. Returns a JSON structure in `cloud_portal_facts`
```
cloud_portal_facts:
  key: Optional query filter, such as "fields=id,name" and where key is a Cloud Volume API URI such as cloud_volumes
```

### lcm manage
Manages the Linux Connection Manager required to connect Cloud Volumes to a host.
```
cloud_lcm_state: absent or present
cloud_lcm_installer: Optional URL to installer, will grab whatever is set in default currently otherwise
```

### array setup
Sets up an array from scratch. The Ansible host needs to be configured with ZeroConf in the same network as the arrays that needs to be setup.
```
nimble_array_serial: The serial number to setup, i.e AF-123456
nimble_array_discovery_interface: Interface on the Ansible host that is connected to the ZeroConf network, defaults to the network interface with a default gateway
nimble_array_config: Advanced YAML structure listing arrays to setup by serial number. Please see defaults/main.yml for an example
```

### password update
Sets a new password for a user
```
nimble_password_user: Defaults to what's stored in nimble_group_options.username
nimble_password_new: New password, i.e Password-364
```
**Note**: The current password is stored in nimble_group_password
**Note #2**: It's good practice to store passwords with Ansible Vault

### host facts
Gathers host facts. Not currently used.

## Example Playbooks
Here are some example uses of this role. These are cut and paste from [examples](examples)

## Nimble Storage Array Group Examples
Make sure credentials for NLT and the arrays are accessible in the appropriate variables.

### nimble/sample_install.yml
Installs NLT.
```
---
# Provide nimble_linux_toolkit_bundle as an extra var to ansible-playbook, i.e:
# $ ansible-playbook -l limit -e nimble_linux_toolkit_bundle=/tmp/nlt_installer-2.0-0 sample_install.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, nimble_object: nlt, nimble_operation: manage, nimble_linux_toolkit_state: running }
```

### nimble/sample_uninstall.yml
Uninstalls NLT.
```
---
- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, nimble_object: nlt, nimble_operation: manage, nimble_linux_toolkit_state: absent }
```


### nimble/sample_debug.yml
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

### nimble/sample_group_fact.yml
Lightweight debugging.
```
---
- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, nimble_object: group, nimble_operation: facts, nimble_group_facts: { volumes: 'fields=name,serial_number' } }
  tasks:
    - debug: var=nimble_group_facts
```

### nimble/sample_provision.yml
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

### nimble/sample_snapshot.yml
```
---
# Provide nimble_volume and nimble_volume_snapshot as extra vars, i.e:
# $ ansible-playbook -l limit -e nimble_volume=myvol1 -e nimble_volume_snapshot=mysnap1 sample_snapshot.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, nimble_object: volume, nimble_operation: snapshot }
```
### nimble/sample_restore.yml
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


### nimble/sample_resize.yml
```
---
# Provide nimble_volume and nimble_volume_size as extra vars, i.e:
# $ ansible-playbook -e limit -e nimble_volume=myvol1 -e nimble_volume_size=2000 sample_resize.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, nimble_object: volume, nimble_operation: resize }
```

### nimble/sample_decommision.yml
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

### nimble/sample_array_setup.yml
Sets up and array from scratch over the network with ZeroConf.
```
---
# Provide nimble_array_serial as an extra variable and store config structure in host_vars/ansible_host
# $ ansible-playbook -l limit -e nimble_array_serial=XX-123456 sample_array_setup.yml
- hosts: ansible_host
  roles:
    - { role: NimbleStorage.Ansinimble,
        nimble_object: array,
        nimble_operation: setup }
```

### nimble/sample_password_change.yml
Changes the password of a user.
```
---
# Provide nimble_password_new and nimble_password_user as extra vars
# $ ansible-playbook -l limit -e nimble_password_new=mynewpass-123 -e nimble_password_user=myusername sample_password_change.yml
- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble,
        nimble_object: password,
        nimble_operation: update }
```
**Note:** Changing the password of the that NLT user is currently connected to requires removing and re-adding the group. The NLT dependency for non-host related tasks such as "password update" will be removed in a future version.

### nimble/sample_protect.yml
Protects a Nimble volume.
```
---
# Provide nimble_volume and nimble_volume_collection OR nimble_protection_template as extra vars, i.e:
# $ ansible-playbook -l limit -e nimble_volume=myvol1 -e nimble_protection_template=Retain-30 sample_protect.yml
# $ ansible-playbook -l limit -e nimble_volume=myvol1 -e nimble_volume_collection=myexistingvolcoll sample_protect.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, 
        nimble_object: volume,
        nimble_operation: protect,
      }
```

### nimble/sample_unprotect.yml
Disassociates a Nimble volume from a volume collection.
```
---
# Provide nimble_volume as extra vars, i.e:
# $ ansible-playbook -l limit -e nimble_volume=myvol1 sample_unprotect.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, 
        nimble_object: volume,
        nimble_operation: unprotect
      }
```

### nimble/sample_volcoll_create.ym
Creates a new volume collection.
```
---
# Create volume collection
# $ ansible-playbook -l limit -e nimble_volume_collection=myvolcoll1 -e nimble_protection_template=myprottmpl1 sample_volcoll_create.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, 
        nimble_object: volcoll,
        nimble_operation: create
      }
```

### nimble/sample_volcoll_prune.yml
Prunes all empty volume collections.
```
---
# Prune volume collections without associated volumes
# $ ansible-playbook -l limit sample_volcoll_prune.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, nimble_object: volcoll, nimble_operation: prune }
```

### nimble/sample_prottmpl_create.yml
Creates a new protection template.
```
---
# Create default protection template 
# $ ansible-playbook -l limit -e nimble_protection_template=myprottmpl1 sample_prottmpl_create.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, 
        nimble_object: prottmpl,
        nimble_operation: create
      }
  tags:
    - standalone

# Create protection template with schedules, please see InfoSight REST API docs for full schedule options
# Note: downstream_partner_id will be translated from a human-readable group name to an id
- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, 
        nimble_object: prottmpl,
        nimble_operation: create,
        nimble_protection_schedules: {
          myschedule1: {
            description: "Create a snapshot hourly and retain 48",
            period_unit: "hours",
            period: 1,
            num_retain: 48
          },
          myschedule2: {
            description: "Create a snapshot every 5 minutes and retain 24",
            period_unit: "minutes",
            period: 5,
            num_retain: 24
          },
          myotherschedule: {
            downstream_partner_id: "group-nva2",
            num_retain_replica: 32,
            description: "Create a snapshot daily during weekdays at 1am and retain 16",
            period_unit: "days",
            days: "monday,tuesday,wednesday,thursday,friday",
            at_time: 3600,
            num_retain: 16
          }
        }
      }
```

### nimble/sample_prottmpl_delete.yml
Deletes a proection template.
```
---
# Delete protection template 
# $ ansible-playbook -l limit -e nimble_protection_template=myprottmpl1 sample_prottmpl_delete.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, 
        nimble_object: prottmpl,
        nimble_operation: delete
      }
```

### nimble/sample_nimble_partner.yml
Partners with downstream Nimble array.
```
---
# Marry two arrays and test the connection
#
# Upstream variables defines how to partner the downstream
# Downstream variables defines how to partner upstream
#
# The nimble_partnerships var describes partnerships and should be protected with Ansible vault. Please find variable descriptions above.
# 
# $ ansible-playbook -l limit -e nimble_partner=mypartnership sample_nimble_partner.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, 
        nimble_object: replication,
        nimble_operation: partner
      }
```

### nimble/sample_nimble_divorce.yml
Divorces (deletes) a replication partnership.
```
---
# Divorce an array from another
#
# $ ansible-playbook -l limit -e nimble_partner=mypartner sample_nimble_divorce.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, 
        nimble_object: replication,
        nimble_operation: divorce
      }
```

### nimble/sample_cloud_partner.yml
Partners a Nimble Storage array group with Cloud Volumes.
```
---
# Establish partnership with HPE Cloud Volumes
# 
# $ ansible-playbook -l limit -e cloud_store_region -e cloud_store=MyReplicationStore sample_cloud_partner.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, 
        nimble_object: cloud,
        nimble_operation: partner
      }
```

### nimble/sample_cloud_divorce.yml
Divorces a Nimble Storage array group from Cloud Volumes (leaves Replication Store intact)
```
---
# Break partnership with HPE Cloud Volumes Replication Store (leaves store intact)
# 
# $ ansible-playbook -l limit -e cloud_store=MyReplicationStore sample_cloud_divorce.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, 
        nimble_object: cloud,
        nimble_operation: divorce
      }
```

### nimble/sample_store_create.yml
Creates a Replication Store in Cloud Volumes.
```
---
# Creates a replication store of given size.
# Mind that creating Replication Stores does not assume any cloud instances and are done in the context of an arbitray Ansible capable node
# $ ansible-playbook -l limit -e cloud_store=mycloudstore1 -e cloud_store_size=300000 -e cloud_store_region=us-west1 sample_store_create.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, 
        cloud_object: store,
        cloud_operation: create
      }
```

### nimble/sample_store_delete.yml
Deletes a Replication Store in Cloud Volumes.
```
---
# Deletes a replication store
# Mind that deleted Replication Stores does not assume any cloud instances and are done in the context of an arbitrary Ansible capable node
# $ ansible-playbook -l limit -e cloud_store=mycloudstore1 sample_store_delete.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, 
        cloud_object: store,
        cloud_operation: delete
      }
```

### nimble/sample_store_resize.yml
Resizes a Replication Store in Cloud Volumes.
```
---
# Resizes a replication store
# Mind that resizing replication stores does not assume any cloud instances and are done in the context of an arbitrary Ansible capable node
# $ ansible-playbook -l limit -e cloud_store=mycloudstore1 -e cloud_store_size=20000 sample_store_resize.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, 
        cloud_object: store,
        cloud_operation: resize
      }
```

## Cloud Volume Examples
These are some basic examples on how to use this role. Each example assumes credentials variables to HPE Cloud Volumes has been set elsewhere.

### cloud/sample_install.yml
Installs Linux Connection Manager
```
---
- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble,
        cloud_object: lcm,
        cloud_operation: manage,
        cloud_lcm_state: present
      }
```

### cloud/sample_provision.yml
Role stack example on how to provision a new Cloud Volume to an EC2 instance.
```
---
# Provisions a new barebone volume, maps and mounts it. Define cloud_volume in extra variable
# $ ansible-playbook -i ec2.py -l tag_cloudhost -e cloud_volume=mycloudvol1 sample_provision.yml
- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, 
        cloud_object: volume,
        cloud_operation: create,
        cloud_volume_provider: "Amazon AWS",
        cloud_volume_region: "us-west-1",
        cloud_volume_options: {
          private_cloud: "vpc-00000000",
          existing_cloud_subnet: "172.31.0.0/16"
        }
      }
    - { role: NimbleStorage.Ansinimble, 
        cloud_object: volume,
        cloud_operation: map,
        cloud_host_ip: "{{ ansible_default_ipv4.address }}"
      }
    - { role: NimbleStorage.Ansinimble, 
        cloud_object: volume,
        cloud_operation: mount,
        cloud_volume_mountpoint: "/mnt/{{ cloud_volume }}"
      }
```

### cloud/sample_resize.yml
Resizes a Cloud Volume and the host filesystem.
```
---
# Resizes a Cloud Volume, specify parameters as extra variables.
# $ ansible-playbook -i ec2.py -l tag_cloudhost -e cloud_volume=mycloudvol1 -e cloud_volume_size=30000 sample_resize.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, 
        cloud_object: volume,
        cloud_operation: resize,
        cloud_volume_mountpoint: "/mnt/{{ cloud_volume }}"
      }
```

### cloud/sample_update.yml
Sets a new IOPS limit on a Cloud Volume and enables multi-initiator. Full list of keys in `cloud_volume_options` are available in the Cloud Volumes API documentation.
```
---
# Sets an IOPS boundary and makes a Cloud Volume accessible from multiple initators, uses cloud_volume from -e
# $ ansible-playbook -i ec2.py -l tag_cloudhost -e cloud_volume=mycloudvol1 sample_update.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, 
        cloud_object: volume,
        cloud_operation: update,
        cloud_volume_options: {
          iops: 1500,
          multi_initiator: True
        }
      }
```

### cloud/sample_snapshot.yml
Snapshots a Cloud Volume with a named snapshot.
```
---
# Snapshots a Cloud Volume, specifies parametes in extra variables.
# $ ansible-playbook -i ec2.py -l tag_cloudhost -e cloud_volume=mycloudvol1 -e cloud_volume_snapshot=mysnap1 sample_snapshot.yml

- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, 
        cloud_object: volume,
        cloud_operation: snapshot
      }
```

### cloud/sample_restore.yml
Restores a Cloud Volume from a named snapshot.
```
---
# Restore a Cloud Volume, specifies parametes in extra variables.
# $ ansible-playbook -i ec2.py -l tag_cloudhost -e cloud_volume=mycloudvol1 -e cloud_volume_snapshot=mysnap1 sample_restore.yml

- hosts: all
  vars:
    cloud_host_ip: "{{ ansible_default_ipv4.address }}"
  roles:
    - { role: NimbleStorage.Ansinimble, 
        cloud_object: volume,
        cloud_operation: umount
      }
    - { role: NimbleStorage.Ansinimble, 
        cloud_object: volume,
        cloud_operation: unmap,
      }
    - { role: NimbleStorage.Ansinimble, 
        cloud_object: volume,
        cloud_operation: restore
      }
    - { role: NimbleStorage.Ansinimble, 
        cloud_object: volume,
        cloud_operation: map,
      }
    - { role: NimbleStorage.Ansinimble, 
        cloud_object: volume,
        cloud_operation: mount
      }
```

### cloud/sample_decommission.yml
Fully decommisions a Cloud Volume from a host and deletes it.
```
---
# Fully decommisions a Cloud Volume. Define volume in cloud_volume.
# $ ansible-playbook -i ec2.py -l tag_cloudhost -e cloud_volume=mycloudvol1 sample_decommission.yml

- hosts: all
  vars:
    cloud_host_ip: "{{ ansible_default_ipv4.address }}"
  roles:
    - { role: NimbleStorage.Ansinimble, 
        cloud_object: volume,
        cloud_operation: umount,
        cloud_volume_mountpoint: "/mnt/{{ cloud_volume }}"
      }
    - { role: NimbleStorage.Ansinimble, 
        cloud_object: volume,
        cloud_operation: unmap
      }
    - { role: NimbleStorage.Ansinimble, 
        cloud_object: volume,
        cloud_operation: delete
      }
```

### cloud/sample_uninstall.yml
Uninstalls the Linux Connection Manager.
```
---
- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble,
        cloud_object: lcm,
        cloud_operation: manage,
        cloud_lcm_state: absent
      }
```

### cloud/sample_debug.yml
Croaks out all HPE Cloud Volume Portal facts into `cloud_portal_facts`
```
---
- hosts: all
  roles:
    - { role: NimbleStorage.Ansinimble, 
        cloud_object: portal,
        cloud_operation: facts
      }
  tasks:
    - debug: 
        var: cloud_portal_facts
    - debug: 
        var: cloud_host_facts
```

## Security considerations
Always secure your `nimble_group_password` with Ansible Vault to prevent eavesdropping. 

## Contributing
Please feel free to submit pull requests. Include a test in [tests/nimble/test.yml](tests/nimble/test.yml) for local Nimble arrays and [tests/cloud/cloud.yml](tests/cloud/test.yml) if including new functionality. Tests are run manually before merge.
 
## License
Apache 2.0, please see [LICENSE](LICENSE)

## Version
Please see [VERSION](VERSION)

## Author Information
This role is maintained by Michael Mattsson, a HPE Nimble Storage employee, on Nimble Hack Days. Please use the GitHub issue tracker for any queries related to this role.
