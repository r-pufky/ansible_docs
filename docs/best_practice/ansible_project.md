# Ansible Project
Setup Ansible project environment to auto-load on entry with direnv using
environment vars, enabling nested playbook use, GPG vault secrets, and VSCode
support for ansible-lint via ansible.cfg.

## Local Packages

=== "Arch"

    ``` bash
    # Auto configure virtual environments.
    pacman -S direnv libuv
    direnv allow
    ```

=== "Debian"

    ``` bash
    # Auto configure virtual environments.
    curl -LsSf https://astral.sh/uv/install.sh | sh
    apt install direnv
    direnv allow
    ```

## Environment Setup

=== ".gitignore"

    !!! abstract ".gitignore"
        0644 {USER}:{USER}

        ``` bash
        # Ansible
        r_pufky-*.tar.gz
        .ansible/
        .ansible
        .vscode/
        molecule/cache
        COMMIT.md
        TODO.md
        # Auto venv
        .venv/

        # Only include vars from root, not symlinked vars.
        !/host_vars
        !/group_vars
        **/*/host_vars
        **/*/group_vars
        ```

=== ".envrc"

    !!! abstract ".envrc"
        0644 {USER}:{USER}

        ``` bash
        # direnv executes .envrc in bash and exports back to current shell.

        # Link {group,host}_vars to inventory for nested playbook execution.
        link_map=(
          "inventory:.."
          "plays:.."
          "plays/arr:../.."
        )

        for item in "${link_map[@]}"; do
          IFS=':' read -r target link <<< "${item}"

          if [ ! -L "${target}/host_vars" ] && [ -d "host_vars" ]; then
            echo "${target}/host_vars ➔ ${link}/host_vars"
            ln -s "${link}/host_vars" "${target}/host_vars"
          fi
          if [ ! -L "${target}/group_vars" ] && [ -d "group_vars" ]; then
            echo "${target}/group_vars ➔ ${link}/group_vars"
            ln -s "${link}/group_vars" "${target}/group_vars"
          fi
        done

        # Create venv if needed and activate adding ansible testing environment.
        uv sync --all-extras && source .venv/bin/activate
        source ansible.env
        ```

=== "ansible.env"

    !!! abstract "ansible.env"
        0644 {USER}:{USER}

        ``` bash
        ###############################################################################
        # Ansible Collection Test Environment Configuration
        ###############################################################################
        # Create an ansible environment to use. Use direnv to auto-load environment or
        # manually source; then activate venv.
        #
        # Manual:
        #   source ansible.env && source ./venv/bin/activate
        #
        # Direnv:
        #   direnv allow  # only needed one-time.
        #
        # All configuration is done using environment variable per best practice. Check
        # both core and community options when setting or changing values.
        #
        # Generated with:
        #
        #   ansible-config init -t all -f env > /tmp/ansible-env.cfg
        #
        # Environment options are:
        # * Explicitly set.
        # * Not internal-only settings.
        # * Deprecated settings actively removed or noted with a TODO if in use.
        # * Use sh interpretation (0: False, 1: True; etc).
        #
        # Settings can be validated with (errors will be listed):
        #
        #   ansible-config --help
        #   ansible-config dump -t all --only-changed
        #
        # Reference:
        # * https://r-pufky.github.io/ansible_docs/ansible/environment

        ###############################################################################
        # Ansible (Core) Common Options
        ###############################################################################
        # Environment variables for core ansible release.
        #
        # Do not use 'ansible' user (as containers do not have this setup); also do not
        # use custom SSH options as these are not used for podman.
        #
        # Reference:
        # * https://docs.ansible.com/ansible/latest/reference_appendices/config.html

        export ANSIBLE_CONFIG=''  # Do not use config files.
        export ANSIBLE_DISPLAY_SKIPPED_HOSTS=0  # Do not display skipped hosts.
        export ANSIBLE_DUPLICATE_YAML_DICT_KEY='error'  # Explicit duplicate errors.
        export ANSIBLE_STDOUT_CALLBACK='debug'  # ansible-doc -t callback -l.
        export ANSIBLE_USE_PERSISTENT_CONNECTIONS=1  # Persist SSH connections.

        ###############################################################################
        # Ansible (Collection) Environment Options
        ###############################################################################
        # Environment variables for non-core ansible collections.
        #
        # Reference:
        # * https://docs.ansible.com/ansible/latest/collections/environment_variables.html#list-of-collection-env-vars

        export ANSIBLE_ADMIN_USERS='root'  # Only enable root by default.
        export ANSIBLE_PARAMIKO_RECORD_HOST_KEYS=0  # Ignore host keys.
        export ANSIBLE_REMOTE_TMP='/tmp'  # Use RAMFS tmp, not disk (~/.ansible/tmp).
        export ANSIBLE_SYSTEM_TMPDIRS='/tmp'  # Use RAMFS tmp, not disk (/var/tmp).

        ###############################################################################
        # Molecule 25.1.0+ Environment Fix
        ###############################################################################
        # 25.2.0 removed automatic path configuration for molecule tests. Manually set
        # Molecule path resolution during environment setup.
        #
        # Reference:
        # * https://github.com/ansible-community/molecule-plugins/issues/301
        # * https://github.com/ansible/molecule/pull/4380

        export PYTHON_LIB_PATH="$(python3 -c 'import sysconfig; print(sysconfig.get_paths()["purelib"])')"
        export ANSIBLE_FILTER_PLUGINS="${PYTHON_LIB_PATH}/molecule/provisioner/ansible/plugins/filter:${HOME}/.ansible/plugins/filter:/usr/share/ansible/plugins/filter"
        export ANSIBLE_LIBRARY="${PYTHON_LIB_PATH}/molecule/provisioner/ansible/plugins/modules:${PYTHON_LIB_PATH}/molecule_plugins/vagrant/modules:${HOME}/.ansible/plugins/modules:/usr/share/ansible/plugins/modules"
        export ANSIBLE_ROLES_PATH="$(pwd)/roles:${HOME}/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles"

        ###############################################################################
        # libvirt
        ###############################################################################
        # Always use system daemon for VM testing to prevent obtuse issues.
        #
        # Reference:
        # * https://wiki.archlinux.org/title/Libvirt#Configuration

        export LIBVIRT_DEFAULT_URI='qemu:///system'

        ###############################################################################
        # Custom Options
        ###############################################################################
        # Direnv auto-loads from root directory regardless of entrypoint; pwd will
        # always resolve to root dir.

        # Enable ansible commands anywhere. This is especially useful for nested
        # playbooks, allowing for better organization (e.g. plays/group/site.yml).
        #
        # Both locations should report the same variables.
        #
        #   ansible {HOST} -m debug -a "var=hostvars[inventory_hostname]"
        #   cd plays/group; ansible {HOST} -m debug -a "var=hostvars[inventory_hostname]"
        export PROJECT_ROOT=$(pwd)

        export ANSIBLE_INVENTORY="$(pwd)/inventory"
        export ANSIBLE_PRIVATE_KEY_FILE="$(pwd)/certs/ssh/core"
        export ANSIBLE_VAULT_PASSWORD_FILE="$(pwd)/scripts/vault_gpg"

        # Do not warn about control and remote node python version differences.
        export ANSIBLE_PYTHON_INTERPRETER='auto_silent'

        # Include JSON for vaulted files (e.g. traefik secrets).
        export ANSIBLE_YAML_FILENAME_EXT='.yml, .vault, .json'

        # ansible ssh user, Auto-accept new hosts.
        export ANSIBLE_REMOTE_USER='ansible'
        export ANSIBLE_SSH_ARGS='-C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=accept-new -o UserKnownHostsFile=/dev/null'

        # Greatly improves SSH speed, requires 'requiretty' disabled in /etc/sudoers
        # on managed hosts to enable. If issue deploying, disable this first.
        export ANSIBLE_PIPELINING=1
        ```

=== "ansible.cfg"

    !!! abstract "ansible.cfg"
        0644 {USER}:{USER}

        ``` toml
        # ansible.cfg required for ansible-lint. Executing ansible-lint will
        # automatically load this config from the root project directory.
        #
        # These options are explicitly ignored (ANSIBLE_CONFIG='') during environment
        # setup for all other commands using direnv.
        [defaults]
        roles_path = ./roles:~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles
        inventory = inventory
        private_key_file = certs/ssh/core
        vault_password_file = scripts/vault_gpg
        ```

[Redirect ansible caches](../README.md#redirect-ansible-caches) as desired and
[Enable GPG Vault](vault.md).

## Initialize Environment
``` bash
# create and enable the virtual environment. Re-enter to activate.
uv init --bare
uv add ansible ansible-lint argcomplete

direnv allow
```
