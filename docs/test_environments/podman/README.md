# Podman

!!! success "Default Testing Platform"
    Podman is a full systemd container and does not have many of the issues
    (including network issues) that docker does.

    Used to test **all** cases unless there are required bare-metal cases (see
    [vagrant](../vagrant/README.md)).

## Molecule Setup
Standard molecule setup for rootless podman debian container.

!!! abstract "molecule.yml"
    0644 {USER}:{USER}

    ``` yaml
    ---
    dependency:
      name: 'galaxy'
    driver:
      name: 'podman'
    provisioner:
      name: 'ansible'
      config_options:
        defaults:
          interpreter_python: 'auto_silent'  # Suppress warnings.
          callback_whitelist: 'profile_tasks, timer, yaml'  # Display profiling.
          # To cache facts between molecule steps:
          #   fact_caching: 'jsonfile'
          #   fact_caching_connection: '/tmp/facts_cache'
          #   fact_caching_timeout: 7200
          #
          # cache a fact between steps:
          #   ansible.builtin.set_fact:
          #     cacheable: yes
          #     my_fact: "howdy, world"
          #
          # always assert cache is not expired before using and document with args:
          #   - name: 'assert fact_caching not expired'
          #     ansible.builtin.assert:
          #       that:
          #         - ansible_facts['my_fact'] is defined'
          #       fail_msg: 'fact_caching has expired; re-run prepare.'
        ssh_connection:
          pipelining: false  # Does not work with podman.
      # inventory:  # Set all base testing configuration here.
      #   group_vars:
      #     all:
      #       setup_variables: true
      #   host_vars:
      #     {IMAGE}-{TEST}:
      #       setup_variables: true
    platforms:
      - name: '{ROLE}_{TEST}'  # debian_default.
        image: 'ghcr.io/hifis-net/debian-systemd:13'
        systemd: 'always'
        volumes:
          - '/sys/fs/cgroup:/sys/fs/cgroup:ro'
        # capabilities:  # Always comment explicit need (file, command, etc).
        #   - 'SYS_ADMIN'  # Required for systemd systems.
        #   - 'NET_ADMIN'  # Required for network-based testing.
        command: '/lib/systemd/systemd'
        pre_build_image: true
        # privileged: true  # For some operations like sysctl, ulimit.
        published_ports:
          - '3000:3000/tcp'  # Exposes port to host. Not needed for testing.
    verifier:
      name: 'ansible'
    lint: |
      set -e
      yamllint .
      ansible-lint .
    # Disable steps as needed with explicit reasons to minimize warnings.
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

## Reference[^1][^2][^3][^4][^5][^6]
[^1]: https://ansible.readthedocs.io/projects/molecule/guides/systemd-container/
[^2]: https://ansible.readthedocs.io/projects/molecule/examples/podman/
[^3]: https://ansible.readthedocs.io/projects/molecule/configuration/?h=test_sequence#scenario
[^4]: https://stackoverflow.com/questions/65350229/how-do-i-share-variables-facts-between-molecule-playbooks
[^5]: https://www.ansible.com/blog/developing-and-testing-ansible-roles-with-molecule-and-podman-part-1/
[^6]: https://wiki.archlinux.org/title/Podman
