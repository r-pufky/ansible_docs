# Role Developemnt

!!! info "Prerequisites"
    * [Collection Created](../collection/README.md).

## Create New Role

### Add role as submodule
Always add roles from repository root as submodules.

``` bash
git submodule add https://github.com/{USER}/{REPO} roles/{ROLE}
```
Reference:

* https://git-scm.com/book/en/v2/Git-Tools-Submodules

### Create role template files (optional)
Ansible galaxy will overwrite GIT metadata when run directly on the submodule.
Instead write a skeleton role and copy data in as-needed. See
[existing roles](https://github.com/r-pufky/ansible_paperless_ngx).

``` bash
ansible-galaxy role init --init-path /tmp example
```
Reference:

* https://goetzrieger.github.io/ansible-collections/5-creating-collections/#adding-content-adding-a-custom-role

### .gitignore
``` yaml
# Ansible
r_pufky-*.tar.gz
.ansible/
.ansible
.vscode/
molecule/cache
COMMIT.md
TODO.md
```
Reference:

* https://github.com/r-pufky/ansible_paperless_ngx


## Update and Commit
Role must be committed to the repository before a new commit hash can be stored
for the collection.

``` bash
cd roles/{ROLE}
git add .
git commit
git push
git submodule update --init  # Update submodule commit hash reference.
cd {COLLECTION}
git add roles/{ROLE}  # Add updated submodule from collection root.
```
Reference:

* https://git-scm.com/book/en/v2/Git-Tools-Submodules
