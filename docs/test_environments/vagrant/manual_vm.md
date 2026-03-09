# Manual VM Creation

!!! info "Prerequisites"
    * [VirtualBox](../virtualbox/README.md).
    * [Vagrant](../vagrant/README.md).

!!! warning "Extreme Circumstances Only"
    Where full hardware virtualization and manual input are needed to test
    (e.g. secure boot requiring manual steps). Explicit reasons for using this
    test framework **MUST BE** clearly documented in `molecule.yml`.

    These cannot be automated and should always be flag protected. Test should
    pass without flag to ensure `molecule test --all` executes without failure.

## System Images Used

* [debian netinstall iso](https://www.debian.org/CD/netinst/)


## Create VM
Use vagrant-like configuration to maintain similarities with other VM tests.
Always use absolute paths as some options do **not** resolve paths.

### Create and list VM
``` bash
mkdir /home/${USER}/VirtualBox VMs/debian-12-efi-template
vboxmanage list ostypes
vboxmanage createvm \
  --basefolder '/home/${USER}/VirtualBox VMs/debian-12-efi-template' \
  --ostype debian12_64 \
  --default \
  --register \
  --name 'debian-12-efi-template'
vboxmanage list vms
export UUID={UUID}  # Always reference with UUID.
vboxmanage showvminfo ${UUID}
```

### Use bridged networking (if needed)
``` bash
vboxmanage modifyvm ${UUID} --nic1 bridged
vboxmanage modifyvm ${UUID} --bridgeadapter1 {DEVICE}  # Host linux device.
```
Reference:

* https://forums.virtualbox.org/viewtopic.php?t=78292

### Enable UEFI and load certificates
``` bash
# enable EFI. alternatives: efi, efi32.
vboxmanage modifyvm ${UUID} --cpus 2 --memory 4096 --firmware efi64
vboxmanage modifynvram $UUID inituefivarstore  # Reset NVRAM state.
# Load default MS KEK/DB signatures for secure boot.
vboxmanage modifynvram $UUID enrollmssignatures
# Enable default platform keys for secure boot.
vboxmanage modifynvram $UUID enrollorclpk
```
Reference:

* https://www.virtualbox.org/manual/UserManual.html#vboxmanage-modifynvram

### Secure Boot Toggle
Secure boot state cannot be changed with `mokutil` in VM; explicitly set state
to disabled for base testing and enable when needing to test final state.

``` bash
vboxmanage modifynvram $UUID secureboot --disable
```

### Attach Storage
``` bash
vboxmanage createmedium \
  disk --size 6000 \
  --format vdi \
  --filename '/home/${USER}/VirtualBox VMs/debian-12-efi-template/disk.vdi'
vboxmanage storageattach ${UUID} \
  --storagectl 'SATA' \
  --device 0 \
  --port 1 \
  --type hdd \
  --medium '/home/${USER}/VirtualBox VMs/debian-12-efi-template/disk.vdi'
vboxmanage storageattach ${UUID} \
  --storagectl 'SATA' \
  --port 2 \
  --type dvddrive \
  --medium '/home/${USER}/debian-12.8.0-amd64-netinst.iso'
```

### Start VM
``` bash
vboxmanage startvm ${UUID}
```
Reference:

* https://docs.oracle.com/en/virtualization/virtualbox/6.0/user/efi.html


## Base OS Install
* Hostname: **debian**
* Domain: **{EMPTY}**
* users (password):
    * **root** (**vagrant**)
    * **vagrant** (**vagrant**)
* guided partition:
    * **whole disk**
    * **one partition**
    * **overwrite**
* packages:
    * **ssh**
    * **standard system utilities**

Reboot once complete.

### Install vagrant SSH keys
``` bash
wget https://raw.githubusercontent.com/hashicorp/vagrant/refs/heads/main/keys/vagrant.pub -O /root/.ssh/authorized_keys
```
Reference:

* https://github.com/hashicorp/vagrant/tree/main/keys

### Zero disk and clear history
Enables high compression when imaged and a clean system state.

``` bash
apt get clean
dd if=/dev/zero of=/EMPTY bs=1M
rm -f /EMPTY
cat /dev/null > ~/.bash_history && history -c && exit
```
Reference:

* https://gist.github.com/srijanshetty/86aa6540d27736c1c11b

### Shutdown and Image VM
``` bash
VBoxManage controlvm ${UUID} acpipowerbutton
# Backup virtualbox image pristine w/ disk.
vboxmanage export ${UUID} --output exported-debian-12-efi.ovf --ovf20
```
Reference:

* https://www.engineyard.com/blog/building-a-vagrant-box-from-start-to-finish/
* https://andreafortuna.org/2019/10/24/how-to-create-a-virtualbox-vm-from-command-line/

### Manually export image and import into vagrant
Export machine for vagrant use. Does **not** enable automatic VM load unless
image is also published.

``` bash
vagrant package --base ${UUID} --output ucs.box  # Template VM UUID.
# Use 'vagrant-debian' in molecule tests.
vagrant box add ucs.box --name vagrant-debian
```

## Manual VM Molecule Setup
Standard Molecule setup for manual VM (see [Molecule](../molecule/README.md)).

### Copy vagrant SSH keys
Vagrant SSH keys [are published here](https://github.com/hashicorp/vagrant/tree/main/keys).

* `molecule/id.{pub,key}` for use in all molecule tests.
* `molecule/{TEST}/id.{pub,key}` for specific molecule tests.

### Set restricted permissions
Molecule will fail SSH connections and not mention permissions issues.

``` bash
chmod 0400 id.*
```

molecule.yml (1)
{ .annotate }

1. 0644 {USER}:{USER}

``` yaml
provisioner:
  inventory:
    group_vars:
      all:
        ansible_ssh_user: 'root'  # Molecule users logged in user by default.
        ansible_ssh_private_key_file:
          '{{ lookup("env", "MOLECULE_PROJECT_DIRECTORY") }}/molecule/id.key'
        # For key only in test location.
        # ansible_ssh_private_key_file:
        #   '{{ lookup("env", "MOLECULE_EPHEMERAL_DIRECTORY") }}/id.key'
platforms:
  - name: '192.168.1.10'  # VM IP.
```

create.yml (1)
{ .annotate }

This is an exact copy of default create with provisioning disabled.

1. 0644 {USER}:{USER}

``` yaml
---
- name: 'Create'
  hosts: 'localhost'
  connection: 'local'
  gather_facts: false
  tasks:
    - name: 'Create | molecule inventory'
      ansible.builtin.debug:
        msg: skipping provisioning

    - name: 'Create | populate instance config'
      ansible.builtin.set_fact:
        instance_conf_dict: {'instance': '{{ item.name }}'}
      loop: '{{ molecule_yml.platforms }}'
      register: instance_config_dict
```
Reference:

* https://medium.com/@fabio.marinetti81/validate-ansible-roles-through-molecule-delegated-driver-a2ea2ab395b5
* https://cloudautomation.pharriso.co.uk/post/molecule-vm-create/#:~:text=Molecule%20Configuration,yml
