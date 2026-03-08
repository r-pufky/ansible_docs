# Collection Development

!!! info "Prerequisites"
    * [Ansible Environment](../environment/README.md).
    * [VirtualBox](../../test_environments/virtualbox/README.md).

## Create New Collection

Create new repository with minimal ansible environment
``` bash
git clone https://github.com/{USER}/{REPO} /var/git/{REPO}
cd ~/git/{REPO}
```

### Update Ansible Environment
A minimal (clean) environment is provided to test collection and roles against
a vanilla ansible installation - enforcing only settings which make testing
easier.

!!! abstract "ansible.env"
    0644 {USER}:{USER}

    ``` bash
    ###############################################################################
    # Ansible Collection Test Environment Configuration
    ###############################################################################
    # Create an ansible environment to use. use direnv to auto-load environment or
    # manually source; then activate venv.
    #
    # Manual:
    #   source ansible.env && source /venv/ansible/bin/activate
    #
    # Direnv:
    #   direnv allow  # only needed one-time.
    #   source /venv/ansible/bin/activate
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
    #   source ansible.env
    #   source /venv/ansible/bin/activate
    #   ansible-config --help
    #   ansible-config dump -t all --only-changed
    #
    # Reference:
    # * https://r-pufky.github.io/ansible_collection_docs/ansible/environment

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

Use direnv to automatically load environment on entering directory.

!!! abstract ".envrc"
    0644 {USER}:{USER}

    ``` bash
    dotenv ansible.env
    ```

``` bash
direnv allow  # Auto configure directory on entering.
```


### One-time initialization
Initialization requires the FQCN, however the repository is typically flat.
Create the skeleton template and move to repository base.

``` bash
cd ~/git/{REPO}
source ansible.env
ansible-galaxy collection init --init-path . r_pufky.{COLLECTION}
mv -vn r_pufky/{COLLECTION}/* .
rm -rfv r_pufky
mkdir -p plugins/modules  # Optional for Python modules.
```

!!! tip
    Alternatively copy an existing collection and remove unused options. Be
    sure to update all remaining files.

Reference:

* https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_structure.html#collections-doc-dir
* https://github.com/ansible-collections/collection_template/tree/main
* https://goetzrieger.github.io/ansible-collections/5-creating-collections/

### Enable Submodule Summaries and Out of Date Checks
Check out of date/uncommitted submodules on push to provide a better summary
of submodule status (stored in `.git/config`).

Enable submodule summary on main repo commits/status.
``` bash
git config status.submodulesummary 1
```

Check for submodules out of date/uncommitted when pushing main repo.
``` bash
git config push.recurseSubmodules check
```

## Reference[^1][^2][^3][^4][^5][^6]

[^1]: https://docs.ansible.com/ansible/latest/community/collection_contributors/collection_requirements.html
[^2]: https://docs.ansible.com/ansible/latest/reference_appendices/release_and_maintenance.html#ansible-core-support-matrix
[^3]: https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_structure.html#collection-structure
[^4]: https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html#magic-variables
[^5]: https://ansible.readthedocs.io/projects/lint/rules/
[^6]: https://yamllint.readthedocs.io/en/stable/rules.html
