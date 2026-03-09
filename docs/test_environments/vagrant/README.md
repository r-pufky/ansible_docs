# Vagrant

!!! warning "Explicit Need Only"
    Vagrant VMs are **ONLY** used to test specific cases which cannot be tested
    in containers (kernel, firmware, proc, systemd, networking, etc.) these
    should **never** be default test cases.


## System Images Used

* [debian/trixie64][a]
* [inception-of-things/trixie][b]
* Alternatively a [non-maintained Debian image may be created.][c].

Image repositories:

* https://portal.cloud.hashicorp.com/vagrant/discover
* https://www.osboxes.org/debian


## Molecule Setup
Standard molecule setup for vagrant with virtualbox VM.

!!! abstract "molecule.yml"
    0644 {USER}:{USER}

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

!!! warning "Always **become** ssh.remote_user"
    Always become when creating Molecule tests or setup an ansible user after
    VM turnup to [apply ansible tasks][d].

    !!! abstract "converge.yml"
        0644 {USER}:{USER}

        ``` yaml
        - name: 'Molecule testing step'
          hosts: 'all'
          gather_facts: false
          become: true
        ```


## Reference[^1][^2][^3]

[^1]: https://wiki.archlinux.org/title/Vagrant
[^2]: https://ansible.readthedocs.io/projects/molecule/configuration/?h=test_sequence#scenario
[^3]: https://floatingpoint.sorint.it/blog/post/setting-up-molecule-for-testing-ansible-roles-with-vagrant-and-testinfra


[a]: https://portal.cloud.hashicorp.com/vagrant/discover/debian/trixie64
[b]: https://portal.cloud.hashicorp.com/vagrant/discover/inception-of-things/debian-trixie
[c]: https://raju.dev/building-debian-13-trixie-vagrant-image
[d]: https://stackoverflow.com/questions/33563425/ansible-1-9-4-failed-to-lock-apt-for-exclusive-operation