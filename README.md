# ansible-stuff
Ansible Playbooks, Roles etc for a home Proxmox host and its guest VMs.

## Setup
Install external role dependencies before running any playbook:
```
ansible-galaxy install -r requirements.yml
```

## Playbooks

### proxmox-provisioner
Provisions the Proxmox host itself:
* Unattended-upgrades
* dropbear-luks: unlock the encrypted root over SSH at boot
* Installs Proxmox VE (`lae.proxmox`)
* Network bridge setup
* Host-level GPU passthrough prep

### host-vm-provisioner
Runs against the Proxmox host to build/clone guest VMs:
* Downloads a Debian cloud image and builds a VM template (cloud-init, qemu-guest-agent)
* Clones the template to `vmdeb3` (docker services VM), passing through GPU + scratch NVMe
* Clones the template to `vmdeb-nas` (NAS VM), passing through the NAS HDD
* Applies VM perf tuning (virtio-scsi-single, iothread, multiqueue net) to both clones
* Lowers host `vm.swappiness` to 10 for the VM-hosting workload

### docker-services-vm-provisioner
Provisions `vmdeb3`:
* Unattended-upgrades, i915 firmware, qemu-guest-agent, Docker
* 4G swapfile on `/scratch`
* Mounts the NAS share, and re-exports `/scratch` back over NFS to the workstation
* Generates an SSH key and checks out the docker-stuff config repo from git
* Runs the `cinemagedarr` compose stack via a `docker compose up` systemd timer

### nas-vm-provisioner
Provisions `vmdeb-nas`:
* Unattended-upgrades, qemu-guest-agent
* 4G swapfile on root
* Unlocks the LUKS+LVM data volume, mounts `/data`
* Exports `/data` over NFS to the workstation and `vmdeb3`

### workstation-maintenance
Runs against `localhost`:
* Unattended-upgrades (Pop!_OS/System76 repos wildcarded, same as dietpi)
* autofs NFS automounts
* Flatpak, pyenv, and pip updaters

### dietpi-provisioner
Provisions a DietPi host: bootstraps Python, unattended-upgrades, and unmasks the
`apt-daily` services/timers DietPi disables by default (breaks unattended-upgrades otherwise).

### docker-backup / snapshot-vmdeb3
* `docker-backup`: stops running containers and tars up docker-stuff on `vmdeb3`
* `snapshot-vmdeb3`: runs the same backup, then detaches `vmdeb3`'s GPU/disk passthrough
  and takes a Proxmox snapshot (passthrough devices can't be snapshotted live)
