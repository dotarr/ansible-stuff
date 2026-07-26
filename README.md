# ansible-stuff
Ansible Playbooks, Roles etc

## Setup
Install external role dependencies before running any playbook:
```
ansible-galaxy install -r requirements.yml
```
(see requirements.yml for a note on mirceanton.proxmox_cloudbuntu's broken upstream repo)

Provisions
* Proxmox host
    - Unlocking of encrypted volume over ssh via dropbear-luks
    - Network stuff (bridge, etc)
    - Host-level settings for GPU passthrough
* Host-level provisioning of guests
    - Cloudbuntu pull of Debian vm to make a template
    - Clone of the above to make a usable guest vm passing through gpu+scratch ssd
    - Restore blank OMV dump + passthrough of nas drive
* Provision OMV
    - Copy in LUKS keys, activate share
* Provision Debian guest
    - Download & add i915 firmware
    - Configure gpu passthrough
    - Auto unlock of scratch drive
    - Install docker
    - Pull docker services config from git
    - Set up NAS mounts
