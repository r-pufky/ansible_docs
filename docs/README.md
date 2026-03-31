# Ansible
Ansible development documentation and standards for collection and role
development. Intended for developer use only. All [collections published][a]
follow these guidelines.


## Environment Configuration
!!! info "Decision: Use isolated minimal environments"
    All collections and roles **assume** environment is created and active.

Each collection is configured using **direnv** to automatically setup an
isolated minimal environment to minimize hidden dependencies. Environments are
configured at the collection level and used within the role submodules.

### Local Packages
Local package must be installed to activate environment. Standard testing
infrastructure: ansible, molecule, podman, vagrant, libvirt, uv, direnv.

=== "Arch"

    ``` bash
    # Auto configure virtual environments, podman, vagrant.
    pacman -S direnv libuv podman crun vagrant
    direnv allow

    # libvirt, KVM/QEMU Driver.
    pacman -S libvirt virt-manager qemu-base

    # VM support: NAT/DHCP, IPtables w/ NFtables bridge, netcat for SSH.
    pacman -S dnsmasq iptables-nft openbsd-netcat
    ```

=== "Debian"

    ``` bash
    # Auto configure virtual environments, podman, vagrant.
    curl -LsSf https://astral.sh/uv/install.sh | sh
    apt install direnv podman crun vagrant
    direnv allow

    # libvirtd, virsh & UI Manager.
    apt install libvirt virt-manager libvirt-daemon-system libvirt-clients

    # KVM/QEMU Driver.
    apt install qemu-kvm

    # VM support: NAT/DHCP, IPtables w/ NFtables bridge, netcat for SSH.
    apt install bridge-utils dnsmasq-base iptables-nft
    ```

### Package Configuration

=== "[Rootless Podman][d]"
    An entry must exist for each user that wants to use rootless podman. Modern
    linux distros create users having these entries by default. Create
    [Subordinate UID/GID Mappings if needed.][e]

    ``` bash
    # Confirm rootless support (graphdrivername=overlay and diff=true).
    podman info | grep -i overlay

    > 107:  graphDriverName: overlay
    > 114:    Native Overlay Diff: "true"

    # Verify unprivileged user namespace (1=enabled).
    sysctl kernel.unprivileged_userns_clone

    > kernel.unprivileged_userns_clone = 1

    # Check user/group is correctly mapped.
    cat /etc/subuid | grep {USER}

    > {USER}:100000:65536

    cat /etc/subgid | grep {GROUP}

    > {GROUP}:100000:65536

    # Modern linux distros may use this (or manually add lines above to files).
    usermod --add-subuids 100000-165535 --add-subgids 100000-165535 {USER}
    ```

    Set [Default Container Registry][f]
    !!! abstract "/etc/containers/registries.conf"
        0644 root:root

        ``` ini
        [registries.search]
        registries = ['ghcr.io']
        unqualified-search-registries=['ghcr.io']
        ```

    ``` bash
    # GHCR.IO requires a github login to download images.
    podman login ghcr.io  # github account.
    ```

    Apply configuration changes and [migrate existing containers][g]
    ``` bash
    podman system reset
    podman system migrate
    ```

=== "[Vagrant][h]"

    ``` bash
    # Enable libvirt plugins.
    vagrant plugin install vagrant-libvirt
    ```

    See [fish shell completions for vagrant][q].

=== "[libvirt][i]"
    Enable [default use of libvirtd (system)][n] for users in libvirt group
    without password. Use [PolicyKit][j] for group based access to run VMs.
    Alternatively [use unix sockets][k].

    !!! abstract "/etc/libvirt/libvirtd.conf"
        0644 root:root

        ``` bash
        auth_unix_ro = "polkit"
        auth_unix_rw = "polkit"
        ```

    !!! abstract "/etc/polkit-1/rules.d/50-libvirt.rules"
        0644 root:root

        ``` js
        // Allow users in libvirt group to manage the libvirt system daemon without authentication.
        polkit.addRule(function(action, subject) {
            if (action.id == "org.libvirt.unix.manage" &&
                subject.isInGroup("libvirt")) {
                    return polkit.Result.YES;
            }
        });
        ```
    ``` bash
    # Add user to VM access groups.
    sudo usermod -aG libvirt,libvirt-qemu,kvm $(whoami)

    # Enable libvirtd. PolicyKit changes require reboot.
    systemctl enable --now libvirtd
    reboot
    ```

    Use [qemu:///system][l] as default hypervisor driver.

    !!! abstract "~/.config/libvirt/libvirt.conf"
        0644 root:root

        ``` ini
        # Users will use qemu:///session which runs libvirtd with current user
        # permissions resulting in obtuse networking VM limitations.
        #
        # /etc/libvirt/libvirt.conf is only read in root user context.
        #
        # ansible.env: export LIBVIRT_DEFAULT_URI=qemu:///system
        uri_default = "qemu:///system"
        ```

    ``` bash
    # Confirm system, session, and defaults work.
    virsh -c qemu:///system
    virsh -c qemu:///session
    virsh

    > Welcome to virsh, the virtualization interactive terminal.
    >
    > Type:  'help' for help with commands
    >        'quit' to qui
    >
    > virsh #
    ```

    Use iptables and enable default network.

    !!! info "[libvirtd requires iptables with UFW][m]"
        libvirt switched to using nftables userspace directly. Immediate
        workaround is to set libvirt back to iptables backend.

        UFW uses legacy iptables userspace tools and these create shared
        nftables top level tables. All different applications which use
        iptables userspace end up in same nftables table.

    !!! abstract "/etc/libvirt/network.conf"
        0644 root:root

        ``` ini
        firewall_backend = "iptables"
        ```

    ``` bash
    # Confirm both sudo and non-sudo commands report default network.
    # Both will show information when qemu:///system is loaded correctly with
    # group membership and policy kit.
    sudo virsh net-list --all
    virsh net-list --all

    >  Name      State      Autostart   Persistent
    > ----------------------------------------------
    >  default   inactive   no          yes

    # Set autostart and start default network.
    virsh net-start default
    virsh net-autostart default

    virsh net-list --all

    >  Name      State    Autostart   Persistent
    > --------------------------------------------
    >  default   active   yes         yes
    ```


### Environment Setup
A minimal (clean) environment enforces only settings which make testing easier
for the collection and roles.

!!! warning "Only use for initial collection configuration."
    Commit environment setup after creation and use direnv to automatically
    setup virtual environment on entering directory.

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
        ```

=== ".envrc"

    !!! abstract ".envrc"
        0644 {USER}:{USER}

        ``` bash
        # direnv executes .envrc in bash and exports back to current shell.

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
        ```

``` bash
uv init --bare
uv add ansible ansible-lint argcomplete
uv add molecule molecule-plugins[podman] molecule-plugins[vagrant]

direnv allow
```

## Redirect ansible caches
High IOPs cause premature SSD wear and cause false **yamllint**,
**ansible-lint** errors from [imported collections and roles][b]. Redirection
allows no changes for production environments while minimizing development
wear. This also applies to [libvirt][o] and [Podman][p].

Consider moving and linking entire .cache directories.

!!! abstract "/etc/fstab"
    0644 root:root

    ``` bash
    # Create a ansible cache directory on boot.
    tmpfs /tmp/ansible-cache tmpfs noatime,mode=1777 0 0
    ```

``` bash
# Mount directory.
mount -o remount -a

# Redirect all per-directory ansible caches.
find . -type d -name '.ansible' -exec rm -rf "{}" \; -exec ln -s /tmp/ansible-cache "{}" \;

# Redirect static ansible caches. Delete or move existing cache data.
ln -s /mnt/cache/home/${USER}/.ansible_async ${HOME}/.ansible_async
ln -s /mnt/cache/home/${USER}/.ansible ${HOME}/.ansible
ln -s /mnt/cache/home/${USER}/.cache/ansible-compat ${HOME}/.cache/ansible-compat
ln -s /mnt/cache/home/${USER}/.cache/molecule ${HOME}/.cache/molecule
# Redirect VScode variants as needed.
ln -s "/mnt/cache/home/${USER}/.config/Code - OSS" "${HOME}/.config/Code - OSS"
ln -s /mnt/cache/home/${USER}/.config/VSCodium ${HOME}/.config/VSCodium
ln -s /mnt/cache/home/${USER}/.config/VSCode ${HOME}/.config/VSCode
# Redirect Podman.
ls -s /mnt/cache/home/${USER}/.local/share/containers ${HOME}/.local/share/containers  # graph.
ln -s /mnt/cache/home/${USER}/.cache/containers ${HOME}/.cache/containers  # config.
ln -s /mnt/cache/var/lib/containers/storage /var/lib/containers/storage  # graph.
# Redirect Vagrant.
ls -s /mnt/cache/home/${USER}/.vagrant.d ${HOME}/.vagrant.d
# Redirect libvirt.
ln -s /mnt/cache/var/lib/libvirt /var/lib/libvirt
```


## VSCode
All VSCode variants are included.

### Create VSCode specific Ansible Environment
Ideally pin to the specific ansible release being used.

``` bash
# A separate environment specifically for VSCode is recommended. Any
# pre-configured environment (e.g. collection) will work.
mkdir -p /var/venv/vscode && cd !$
uv init --bare
uv add ansible ansible-lint argcomplete ansible-navigator
uv add molecule molecule-plugins[podman] molecule-plugins[vagrant]
```

Install [Ansible extension][c]. There should be no errors when opening ansible
files locally or remotely.

!!! example "ansible ➔ settings ➔ user ➔ workspace"
    * Ansible: Path ➔ /var/venv/vscode/.venv/bin/ansible
    * Ansible: Use Fully Qualified Collection Names ➔ ✔
    * Ansible: Reuse Terminal ➔ ✔
    * Python: Interpreter Path ➔ /var/venv/vscode/.venv/bin/python
    * Python: Activation Script ➔ /var/git/vscode/.venv/ansible.env
    * Ansible: Navigator Path ➔ ansible-navigator
    * Completion: Provide Redirect Modules ➔ ✔
    * Completion: Provide Module Option Aliases ➔ ✔
    * Validation: Enabled ➔ ✔
    * Lint: Enabled ➔ ✔
    * Lint: Path ➔ ansible-lint
    * Execution Environment: Container Engine ➔ auto
    * Execution Environment: Image ➔ ghcr.io/ansible/community-ansible-dev-tools:latest
    * Pull: Policy ➔ missing
    * Telemetry: Enabled ➔ ✘
    * Lightspeed: Enabled ➔ ✘

!!! example "ctrl + , Python: Venv Path ➔ /var/venv"

!!! tip "Preferences ➔ Color Theme ➔ Dark Modern"

!!! tip ""
    Disable `ansible-lint` for documentation to prevent incorrect YAML error reporting.

    Extensions ➔ Ansible ➔ Settings ➔ Folder
    ``` yaml
    ansible.validation.lint.enabled: false
    ```

    Manually run on the command line.


## Reference[^1][^2]

[^1]: https://docs.ansible.com/ansible/latest/roadmap/ansible_roadmap_index.html
[^2]: https://docs.ansible.com/ansible/2.9/installation_guide/intro_installation.html


[a]: https://galaxy.ansible.com/ui/namespaces/r_pufky
[b]: https://github.com/ansible/ansible-lint/issues/4533
[c]: https://marketplace.visualstudio.com/items?itemName=redhat.ansible
[d]: https://github.com/ansible-community/molecule-podman
[e]: https://github.com/systemd/systemd/issues/21952
[f]: https://halukkarakaya.medium.com/how-to-configure-default-search-registries-in-podman-ea930289692
[g]: https://github.com/containers/podman/issues/12715
[h]: https://github.com/vagrant-libvirt/vagrant-libvirt
[i]: https://libvirt.org
[j]: https://wiki.archlinux.org/title/Polkit
[k]: https://wiki.archlinux.org/title/Libvirt#Authenticate_with_file-based_permissions
[l]: https://wiki.archlinux.org/title/Libvirt#Configuration
[m]: https://gitlab.com/libvirt/libvirt/-/work_items/644
[n]: https://wiki.archlinux.org/title/Libvirt#Installation
[o]: https://old.reddit.com/r/linuxquestions/comments/tkgqdu/how_to_change_storage_path_for_libvirt
[p]: https://docs.podman.io/en/latest
[q]: https://r-pufky.github.io/docs/app/fish/#vagrant-shell-completion