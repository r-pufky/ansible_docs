# Role Development


## New Role
!!! warning "[Ansible galaxy][c] will overwrite GIT metadata when run directly on the submodule"
    Instead write a skeleton role and copy data in as-needed. See [example][b].

    ``` bash
    ansible-galaxy role init --init-path /tmp example
    ```

!!! tip
    Always add roles from repository root [as submodules][a].

``` bash
git submodule add https://github.com/{USER}/{REPO} roles/{ROLE}
```

!!! abstract ".gitignore"
    0644 {USER}:{USER}

    ``` bash
    # Ansible
    .ansible/
    .ansible
    .vscode/
    molecule/cache
    COMMIT.md
    TODO.md
    ```

Redirect [ansible caches](README.md#redirect-ansible-caches).


## Commits
Role must be committed before a [new commit hash][d] can be stored in the
collection.

``` bash
# Standard git commit example.
cd roles/{ROLE}
git add .
git commit
git push

# Update submodule commit hash reference. This does not overwrite repo state.
git submodule update --init

# Add updated submodule from collection root.
cd {COLLECTION}
git add roles/{ROLE}
```

[a]: https://git-scm.com/book/en/v2/Git-Tools-Submodules
[b]: https://github.com/r-pufky/ansible_paperless_ngx
[c]: https://goetzrieger.github.io/ansible-collections/5-creating-collections/#adding-content-adding-a-custom-role
[d]: https://git-scm.com/book/en/v2/Git-Tools-Submodules