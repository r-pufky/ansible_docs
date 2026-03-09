# Collection Development

!!! info "Prerequisites"
    * [VirtualBox](../test_environments/virtualbox/README.md).

## Create New Collection

Create new repository with minimal ansible environment
``` bash
git clone https://github.com/{USER}/{REPO} /var/git/{REPO}
cd ~/git/{REPO}
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
