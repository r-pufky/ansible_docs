# Commit Checks

!!! info "Prerequisites"
    * [Ansible Environment](../environment/README.md).

## Ensure all TODOs are valid
``` bash
grep -ri todo
```

## Ensure linting follows [guidelines](../role/style_guide.md)
``` bash
grep -ri yamllint  # yamllint
grep -ri noqa  # ansible-lint
```

## Lint returns clean
``` bash
yamllint .
ansible-lint .
```
Reference:

* https://yamllint.readthedocs.io/en/stable/
* https://ansible.readthedocs.io/projects/lint/rules/

## Add collection ignore files
Add files that should not be included in the built collection such as tests.

galaxy.yml
``` yaml
# Do not include: testing, environments, caches, IDE settings, or staging
# messages in release tarball.
build_ignore:
  - '*.ansible'
  - '*.gitignore'
  - '*.gitmodules'
  - '*.git'
  - '*molecule'
  - '.envrc'
  - '.vscode'
  - 'ansible.env'
  - 'TODO.md'
  - 'COMMIT.md'
```
Reference:

* https://docs.ansible.com/ansible/devel/dev_guide/developing_collections_distributing.html#ignoring-files-and-folders

## Update collection submodule reference
Required otherwise the collection will not use the updated module on checkout.

[See submodule Updates and Commit](../role/README.md#update-and-commit)

## Versioning
Only tag versions when a new release is ready.
[See Major Release Guide](release.md#versioning).

## Track Project Updates
* Always link to the latest release (or release page).
* Set alerts to check for new release (or push notifications) for each role.

## Build Galaxy Release
``` bash
ansible-galaxy collection build
ansible-galaxy collection publish  # Alternatively manually upload to Galaxy.
```
Reference:

* https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_creating.html
