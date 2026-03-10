# Troubleshooting

!!! info ""
    See [molecule](../molecule/troubleshooting.md) for specific molecule
    troubleshooting.


## Driver vagrant does not [provide a schema][a]
Vagrant driver does have a schema and will always generate a warning.

!!! danger ""
    ``` log
    WARNING Driver vagrant does not provide a schema.
    ```


## A Vagrant environment or target machine is required to run this command
Virtualbox kernel modules are likely not loaded.

!!! danger ""
    ``` log
    A Vagrant environment or target machine is required to run this
    command. Run 'vagrant init' to create a new Vagrant environment. Or,
    get an ID of a target machine from 'vagrant global-status' to run
    this command on. A final option is to change to a directory with a
    Vagrantfile and to try again.
    ```

Install kernel modules.
=== "Arch"

    ``` bash
    pacman -Q | grep linux
    pacman -S linux-{KERNEL}-virtualbox-host-modules
    systemctl reboot
    ```

=== "Debian"

    ``` bash
    dpkg -l | grep linux-image | grep ii
    apt install linux-image-amd64 linux-headers-amd64
    systemctl reboot
    ```

``` bash
# Confirm each command works.
vboxmanage --version  # Executable with modules loaded.
> 7.1.4r165100  # Installed version.
vagrant global-status
vagrant up --provider virtualbox  # Provides details why it is not running.
```

## VERR_CFGM_NOT_ENOUGH_SPACE
Molecule test fails during create due to vagrant path length limitation (see
underlying virtualization manager).

!!! danger ""
    ``` bash
    fatal: [localhost]: FAILED! => {
        "changed": false,
        "cmd": [
            "/usr/bin/vagrant",
            "up",
            "--no-provision"
        ],
        "rc": 1
    }

    STDERR:

    ### YYYY-MM-DD 20:32:26 ###
    ### YYYY-MM-DD 20:32:26 ###
    There was an error while executing `VBoxManage`, a CLI used by Vagrant
    for controlling VirtualBox. The command and stderr is shown below.

    Command: ["startvm", "b3415658-8f60-4026-8375-f6ec52014db5", "--type", "headless"]

    Stderr: VBoxManage: error: Failed to construct device 'ichac97' instance #0 (VERR_CFGM_NOT_ENOUGH_SPACE)
    VBoxManage: error: Details: code NS_ERROR_FAILURE (0x80004005), component ConsoleWrap, interface IConsole

    MSG:

    Failed to start the VM(s): See log file '.../vagrant.err'
    ```

!!! info "Molecule creates a VM name using"
    * Molecule test name: **my_test**
    * Molecule image test name: **test-case-vm**
    * Image box name: **inception-of-things-trixie**
    * OS used: **debian**
    * Network: **private_network**
    * Random UUID hash: **36 characters**

Use shorter component names:

* VM instance name in **molecule.yml**.
* Rename molecule test to shorter name.
* Extreme cases may require cloning repo with a shorter root repository name.

my_role/molecule/**my_test**/molecule.yml
``` yaml
platforms:
  - name: 'test-case-vm'
    box: 'inception-of-things/trixie'
    interfaces:
      - network_name: 'private_network'
```


## Failed to [lock apt for exclusive operation][b]
Debian VirtualBox VM uses root to configure and test the instance. All Molecule
setup/teardown steps require root user.

``` yaml
- name: 'Molecule testing step'
  hosts: 'all'
  gather_facts: false
  # Always **become** ssh.remote_user when creating Molecule tests or setup an
  # ansible user after VM turnup to apply ansible tasks.
  become: true
```


## Failed to start the VM(s)
Virtualbox may fail due to system constraints.

!!! danger ""
    ``` bash
    PLAY [Create] ******************************************************************
    fatal: [localhost]: FAILED! => {
        "changed": false,
        "cmd": [
            "/usr/bin/vagrant",
            "up",
            "--no-provision"
        ],
        "rc": 1
    }

    STDERR:

    ### YYYY-MM-DD 15:10:23 ###
    ### YYYY-MM-DD 15:10:23 ###
    ### YYYY-MM-DD 15:10:23 ###
    ### YYYY-MM-DD 15:32:14 ###
    ### YYYY-MM-DD 15:32:15 ###
    ### YYYY-MM-DD 15:32:15 ###
    The SSH connection was unexpectedly closed by the remote end. This
    usually indicates that SSH within the guest machine was unable to
    properly start up. Please boot the VM in GUI mode to check whether
    it is booting properly.

    MSG:

    Failed to start the VM(s): See log file '.../vagrant.err'
    ```

``` bash
# Re-run test.
molecule test --scenario-name={TEST}
```

## Flaky VM
A previous molecule test likely not cleaned up properly.

``` bash
# Clean all VMs and re-run.
vagrant global-status
vagrant destroy {ID}
vagrant box list
vagrant box destroy {ID}
```

!!! tip "Check GUI for additional VMs if needed."


## ERROR! couldn't [resolve module/action 'vagrant'][c]
Molecule **25.2.0+** is a bad release that broke vagrant modules.

!!! danger ""
    ``` bash
    molecule create
    > WARNING  Driver vagrant does not provide a schema.
    > INFO     default scenario test matrix: dependency, create, prepare, converge
    > INFO     Performing prerun with role_name_check=0...
    > INFO     Running default > dependency
    > WARNING  Skipping, missing the requirements file.
    > WARNING  Skipping, missing the requirements file.
    > INFO     Running default > create
    > ERROR! couldn't resolve module/action 'vagrant'. This often indicates a misspelling, missing collection, or incorrect module path.
    >
    > The error appears to be in '.../molecule_plugins/vagrant/playbooks/create.yml': line 8, column 7, but may
    > be elsewhere in the file depending on the exact syntax problem.
    >
    > The offending line appears to be:
    >
    >   tasks:
    >     - name: Create molecule instance(s) # noqa fqcn[action]
    >       ^ here
    ```

!!! warning "Install Molecule **25.1.0** as a temporary workaround."
    This is only a temporary solution until alternatives are found. Molecule
    developers quite frankly shit out a half-form glob, tosseed it over the
    fence, yelled fuck you, and walked away. Most consumers of Molecule have
    locked to 25.1 as the workarounds are ineffective or created a [ton of
    unnecessary churn][c].

    This is a serious breakage with an extreme lack of reponse from developers.

    ``` bash
    pip install --force-reinstall -v 'molecule==25.1.0'
    ```

!!! info "Potential Resolutions"
    * [Main issue][c]
    * [direnv workaround][d]
    * [Developers abandoning Molecule updates][e]
    * [Developer merged pull][f]
    * [Potential workaround][g]
    * [Developer response][h]
    * [Script to generate previous molecule environment][i]
    * [opentofu stopped molecule updates][j]


# TODO - update with default user/pass, ssh user/pass, and links to public keys.


debian package: root/vagrant.  SSH only with vagrant/vagrant user w/ public key.


## Permission denied (publickey,password)
Verify permissions are correct and keys are loaded from the correct location.
Applies to [Manual VM](manual_vm.md) only.

!!! danger ""
    ``` bash
    TASK [Install packages] ********************************************************
    included: {ROLE} for 10.2.2.39
    fatal: [10.2.2.39]: UNREACHABLE! => {
        "changed": false,
        "unreachable": true
    }
    MSG:
    root@10.2.2.39: Permission denied (publickey,password).
    ```

``` bash
# Confirm keys are set correctly in VM.
ls -l id.*
> 0400 {USER}:{USER} id.*
```

!!! abstract "converge.yml"
    0644 {USER}:{USER}

    ``` yaml
    # Add debug line in converge to confirm key is located correctly
    - ansible.builtin.debug:
        msg: '{{ ansible_ssh_private_key_file }}'
    ```

## VM moved or not registered
VMs must be registered to start in VirtualBox. Applies to
[Manual VM](manual_vm.md) only.

``` bash
vboxmanage registervm \
  /home/{USER}/VirtualBox\ VMs/debian-12-efi-template/debian-12-efi-template.vbox
```

[a]: https://github.com/ansible/molecule/discussions/4108
[b]: https://stackoverflow.com/questions/33563425/ansible-1-9-4-failed-to-lock-apt-for-exclusive-operation
[c]: https://github.com/ansible-community/molecule-plugins/issues/301
[d]: https://github.com/ansible-community/molecule-plugins/issues/301#issuecomment-3687957567
[e]: https://github.com/dev-sec/ansible-collection-hardening/pull/849
[f]: https://github.com/ansible/molecule/pull/4380
[g]: https://github.com/cisagov/skeleton-ansible-role/pull/221/changes
[h]: https://docs.ansible.com/projects/dev-tools/user-guide/test-isolation
[i]: https://github.com/konstruktoid/ansible-role-hardening/blob/master/generate_molecule_env.sh
[j]: https://github.com/diodonfrost/ansible-role-opentofu/commit/b06e2850c4dea0054570033d231bc35c28d7245b
