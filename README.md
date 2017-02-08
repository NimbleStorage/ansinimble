# ansinimble
Roles and playbooks for mnemonic devices for humans with amnesia 

## Intro
This repo contains reusable Ansible roles and playbooks to manage Nimbe Storage infrastructure and surrounding applications.

## Assumptions and prerequisites
These roles assumes the latest Nimble Linux Toolkit (NLT) is installed on the host being managed by Ansible. The latest version of NLT may be obtained from [InfoSight](https://infosight.nimblestorage.com) (Nimble customers and partners only).

* All roles assumes that the userland XFS userland utilities are installed
* Some roles may not require super user privileges but most do
* Requires Ansible 2.2+ with the jmespath Python package installed

## Roles
Every role have a set of host variables assigned depending on the operation. Some are mandatory but most are optional and are set to sane defaults.

* [nimble_host_facts](roles/nimble_host_facts) - Gathers facts about a host system
* [nimble_group_facts](roles/nimble_group_facts) - Gathers facts about Nimble array group connected to the host system
* [nimble_volume_create](roles/nimble_volume_create) - Creates a Nimble volume, either a new one or from a clone
* [nimble_volume_map](roles/nimble_volume_map) - Maps, formats and mounts a Nimble volume to the host
* [nimble_volume_unmap](roles/nimble_volume_unmap) - Unmounts, unmaps and offline a Nimble volume from the host
* [nimble_volume_restore](roles/nimble_volume_restore) - Restores a Nimble volume from a snapshot, requires the volume to be unmapped
* [nimble_volume_delete](roles/nimble_volume_delete) - Deletes a Nimble volume (needs to be unmapped first)
* [nimble_volume_snapshot](roles/nimble_volume_snapshot) - Creates a manual snapshot of a Nimble volume
* [nimble_volume_resize](roles/nimble_volume_resize) - Resizes a Nimble volume and expands the filesystem (online)

## Playbooks
Each role have an example snippet on how to craft a playbook utilizing the particular role.

## Global variables

## Consistentcy

## Security

## License
