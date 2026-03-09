# Testing

!!! info "Prerequisites"
    * [VirtualBox](../virtualbox/README.md).
    * [Vagrant](README.md).

## System Images Used

!!! warning "Licensing"
    Debian team has stopped creating official images due to licensing issues
    with hashicorp.

* [inception-of-things/trixie](https://portal.cloud.hashicorp.com/vagrant/discover/inception-of-things/debian-trixie)
* [boxen/debian-13](https://portal.cloud.hashicorp.com/vagrant/discover/boxen/debian-13) (1)
    { .annotate }

    1. Alternate - currently SSH key build issues.

## Image repositories
* https://portal.cloud.hashicorp.com/vagrant/discover/
* https://www.osboxes.org/debian/

Alternatively a [non-maintained Debian image may be created.](https://raju.dev/building-debian-13-trixie-vagrant-image/)

## Molecule Setup
Standard molecule setup for vagrant with virtualbox VM.

molecule.yml (1)
{ .annotate }

1. 0644 {USER}:{USER}

``` yaml
---
dependency:
  name: 'galaxy'
driver:
  name: 'vagrant'
  provider:
    name: 'virtualbox'
    enable_efi: true
    # provider_raw_config_args:
    #   - "customize [ 'modifyvm', :id, '--firmware', 'efi' ]"
    config_options:
      ssh.keep_alive: true
      ssh.remote_user: 'root'  # Vagrant login user (check VM image).
    options:
      append_platform_to_hostname: false
provisioner:
  name: 'ansible'
  config_options:
    defaults:
      interpreter_python: 'auto_silent'  # Suppress warnings.
      callback_whitelist: 'profile_tasks, timer, yaml'  # Display profiling.
  # inventory:  # Set all base testing configuration here.
  #   group_vars:
  #     all:
  #       setup_variables: true
  #   host_vars:
  #     {IMAGE}-{TEST}:
  #       setup_variables: true
platforms:
  - name: '{ROLE}-{IMAGE}-vm-{TEST}'
    box: 'inception-of-things/trixie'
    memory: 4096
    cpus: 2
    interfaces:
      - network_name: 'private_network'  # Required.
        auto_config: true
        type: 'dhcp'
        # type: static
        # ip: 192.168.56.10  # default is 192.168.56.0/21
    instance_raw_config_args:
      - 'vm.network "forwarded_port", guest: 8443, host: 8443'
      - 'vm.network "forwarded_port", guest: 8080, host: 8080'
      - 'vm.network "forwarded_port", guest: 8880, host: 8880'
      - 'vm.network "forwarded_port", guest: 8443, host: 8843'
verifier:
  name: 'ansible'
# Disable testing steps as needed with explicit reasons to minimize warnings.
scenario:
  test_sequence:
    - 'dependency'
    - 'cleanup'
    - 'destroy'
    - 'syntax'
    - 'create'
    - 'prepare'
    - 'converge'
    - 'idempotence'
    - 'side_effect'
    - 'verify'
    - 'cleanup'
    - 'destroy'
```
Reference:

* https://ansible.readthedocs.io/projects/molecule/configuration/?h=test_sequence#scenario
* https://floatingpoint.sorint.it/blog/post/setting-up-molecule-for-testing-ansible-roles-with-vagrant-and-testinfra

converge.yml (1)
{ .annotate }

1. 0644 {USER}:{USER}

``` yaml
- name: 'Molecule testing step'
  hosts: 'all'
  gather_facts: false
  # Always **become** ssh.remote_user when creating Molecule tests or setup an
  # ansible user after VM turnup to apply ansible tasks.
  become: true
```
Reference:

* https://stackoverflow.com/questions/33563425/ansible-1-9-4-failed-to-lock-apt-for-exclusive-operation
