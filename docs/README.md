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
Local package must be installed to activate environment. Standard Ansible
testing infrastructure: molecule, podman, vagrant, uv, direnv.

=== "Arch"

    ``` bash
    # Auto configure virtual environments.
    pacman -S direnv libuv podman crun vagrant
    direnv allow
    ```

=== "Debian"

    ``` bash
    curl -LsSf https://astral.sh/uv/install.sh | sh
    apt install direnv podman crun vagrant
    direnv allow
    ```

#### Package Configuration

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


### Environment Setup
A minimal (clean) environment enforces only settings which make testing easier
for the collection and roles.

!!! warning "Only use for initial collection configuration."
    Commit environment setup after creation and use direnv to automatically
    setup virtual environment on entering directory.

``` bash
uv init --bare
uv add ansible ansible-lint argcomplete
uv add molecule molecule-plugins[podman] molecule-plugins[vagrant]

direnv allow
```

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

!!! abstract ".envrc"
    0644 {USER}:{USER}

    ``` bash
    # direnv executes .envrc in bash and exports back to current shell.

    # Create venv if needed and activate adding ansible testing environment.
    uv sync --all-extras && source .venv/bin/activate
    source ansible.env
    ```

??? abstract "ansible.env"
    0644 {USER}:{USER}

    ``` bash
    ###############################################################################
    # Ansible Collection Test Environment Configuration
    ###############################################################################
    # Create an ansible environment to use. use direnv to auto-load environment or
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
    ```

###

## Redirect ansible caches
High IOPs cause premature SSD wear and cause false **yamllint**,
**ansible-lint** errors from [imported collections and roles][b]. Redirection
allows no changes for production environments while minimizing development
wear. Consider moving and linking entire .cache directories.

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
ln -s /mnt/cache/.ansible_async ${HOME}/.ansible_async
ln -s /mnt/cache/.ansible ${HOME}/.ansible
ln -s /mnt/cache/cache/ansible-compat ${HOME}/.cache/ansible-compat
ln -s /mnt/cache/cache/molecule ${HOME}/.cache/molecule
# Redirect VScode variant as needed.
ln -s "/mnt/cache/config/Code - OSS" "${HOME}/.config/Code - OSS"
ln -s /mnt/cache/config/VSCodium ${HOME}/.config/VSCodium
ln -s /mnt/cache/config/VSCode ${HOME}/.config/VSCode
# Redirect Podman.
ls -s /mnt/cache/local/share/containers ${HOME}/.local/share/containers  # graph.
ln -s /mnt/cache/cache/containers ${HOME}/.cache/containers  # config.
ln -s /mnt/cache/storage /var/lib/containers/storage  # graph.
```


## VSCode
All VSCode variants are included.

Install [Ansible extension][c].

!!! example "ansible ➔ settings ➔ user ➔ workspace"
    * Ansible: Path ➔ /var/venv/{REPO}/bin/ansible
    * Ansible: Use Fully Qualified Collection Names ➔ ✔
    * Ansible: Reuse Terminal ➔ ✔
    * Python: Interpreter Path ➔ /var/venv/{REPO}/bin/python
    * Python: Activation Script ➔ /var/git/{REPO}/ansible.env
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