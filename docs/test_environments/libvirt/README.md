# [libvirt][a]

!!! warning "Explicit Need Only"
    VirtualBox VMs are **ONLY** used to test specific cases which cannot be
    tested in containers (kernel, firmware, proc, systemd, networking, etc.)
    these should **never** be default test cases.


# Molecule Setup
Standard molecule setup for vagrant libvirt kvm/qemu debian VM.

!!! abstract "molecule.yml"
    0644 {USER}:{USER}

    ``` yaml
    dependency:
      name: 'galaxy'
    driver:
      name: 'vagrant'
      provider:
        name: 'libvirt'
        type: 'libvirt'
    provisioner:
      name: 'ansible'
      connection_options:
        ansible_ssh_user: 'vagrant'
        ansible_become: true
      config_options:
        defaults:
          interpreter_python: 'auto_silent'
          callback_whitelist: 'profile_tasks, timer, yaml'
    platforms:
      - name: 'firmware-default'  # _ not allowed.
        box: 'debian/trixie64'
        memory: 4096
        cpus: 2
        interfaces:
          - network_name: 'private_network'  # network_name required.
            auto_config: true
            type: 'dhcp'
            # type: static
            # ip: 192.168.56.10  # default is 192.168.56.0/21
        instance_raw_config_args:
          - 'vm.network "forwarded_port", guest: 8443, host: 8443'
          - 'vm.network "forwarded_port", guest: 8080, host: 8080'
          - 'vm.network "forwarded_port", guest: 8880, host: 8880'
          - 'vm.network "forwarded_port", guest: 8443, host: 8843'
        # Example provides GPU passthrough.
        # provider_options:
        #   video_type: 'virtio'
        #   channel:
        #     type: 'unix'
        #     target_name: "org.qemu.guest_agent.0"
        #     target_type: "virtio"
        # provider_raw_config_args:
        #   - "cpu_mode = 'host-passthrough'"
        #   - "disk_bus = 'virtio'"
        #   - "nic_model_type = 'virtio'"
    verifier:
      name: 'ansible'
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


## Reference[^1][^2][^3][^4]

[^1]: https://oneuptime.com/blog/post/2026-02-21-how-to-configure-molecule-with-vagrant-driver/view
[^2]: https://github.com/ansible-community/molecule-vagrant/blob/main/README.rst
[^3]: https://wiki.archlinux.org/title/Virt-manager
[^4]: https://wiki.archlinux.org/title/QEMU


[a]: https://wiki.archlinux.org/title/Libvirt