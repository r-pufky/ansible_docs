# Collection Development


## New Collection

!!! tip
    Recommend copying an existing collection, updating, and removing unused
    options.

1. Create a repository and clone it.
2. Enable submodule summaries and out of date checks.

    ``` bash
    # Check out of date/uncommitted submodules on push to provide a better
    # summary of submodule status.

    # Enable submodule summary on main repo commits/status.
    git config status.submodulesummary 1

    # Check for submodules out of date/uncommitted when pushing main repo.
    git config push.recurseSubmodules check
    ```

3. [Generate][a] collection [structure][b].

    ``` bash
    # One-time initialization requires FQCN, however the repository is
    # typically flat. Create and move to repository base.
    cd ~/git/{REPO}
    source ansible.env
    ansible-galaxy collection init --init-path . r_pufky.{COLLECTION}
    mv -vn r_pufky/{COLLECTION}/* .
    rm -rfv r_pufky
    mkdir -p plugins/modules  # Optional for Python modules.
    ```


## Testing
Use community references for examples on specific implementations:

* [Filter Plugins][c]
* [Integration Tests][d]

### Build and Install
The collection must be built and installed locally to be tested.

!!! warning "Collection must be rebuilt if changes (files or tests) were made."
    Any change requires rebuilding, installing **AND** re-entering directory as
    inodes change.

``` bash
# Package auto versions based on galaxy.yml.
ansible-galaxy collection build -f  # -f (force) useful for repeated testing.

# Package is installed to local cache:
# ~/.ansible/collections/ansible_collections/{USER}/{COLLECTION}
ansible-galaxy collection install -f  # -f (force) useful for repeated testing.
```

### Running Tests
Always [build and install](#build-and-install) before running tests.

``` bash
# Always re-enter between installs - cd ~; cd -
cd ~/.ansible/collections/ansible_collections/{USER}/{COLLECTION}
ansible-test {unit,integration,sanity}
```


## Commits
!!! tip "Decision: Use aggressive deprecation"
    Supporting (OS) * (OS versions) * (ansible versions) * (python versions)
    versions exponentially grows and cannot be maintained.

    Pick versions to develop against and lock against those for longer periods
    of time.

    Use release table and enable consumers unable to migrate the ability to
    pickup the burden of maintaining it themselves.


1. Verify [exceptions](../best_practice/style_guide.md) are accurate.

    ``` bash
    # Remove debug messaging as needed.
    grep -ri fail:
    grep -ri debug:
    grep -ri msg:

    # Only accept valid lint exceptions.
    grep -ri todo
    grep -ri yamllint  # yamllint
    grep -ri noqa  # ansible-lint

    # Lint returns clean
    yamllint .
    ansible-lint .
    ```

    [yamllint][e] and [ansible-lint][f] documentation.

2. Update [submodules](../role.md#commits).
3. Update [**galaxy.yml**][g].

    !!! abstract "galaxy.yml"
        0644 {USER}:{USER}

        ``` yaml
        # Always update version, links, and dependencies.

        # Do not include: testing, environments, caches, IDE settings, or staging
        # messages in release tarball.
        build_ignore:
          - '*.ansible'
          - '*.gitignore'
          - '*.gitmodules'
          - '*.git'
          - '*molecule'
          - '.envrc'
          - '.venv'
          - '.vscode'
          - 'ansible.env'
          - 'pyproject.toml'
          - 'uv.lock'
          - 'TODO.md'
          - 'COMMIT.md'
        ```

4. Update **changelogs/changelog.yaml**.

    !!! tip ""
        Truncate changelog and use **ancestor: {LAST VERSION}** to link to
        previous version changelog.

5. Use [Semantic Versioning 2.0.0][h] and provide a [release table][j] in
    README.md.
6. Update links to latest release.
7. Set alerts to check for new release (or push notifications) for each role.


## [Release][k]

!!! tip "Release low-level dependencies first"
    data ➔ deb ➔ srv ➔ arr ➔ game ➔ docs

1. Build & check release.

    ``` bash
    # Sanity check size/contents for unwanted inclusions.
    ansible-galaxy collection build
    ```

2. Upload new artifact to https://galaxy.ansible.com.

    Confirm that collection markdown is formatted correctly (Ansible uses a
    subset of GFM to render documents and may differ from Github). Rebuild and
    re-upload as needed.

    ``` bash
    # Alternatively manually upload to Galaxy.
    ansible-galaxy collection publish
    ```

2. Push github release.


## Reference[^1][^2][^3][^4][^5][^6][^7][^8][^9][^10][^11]

[^1]: https://docs.ansible.com/ansible/latest/community/collection_contributors/collection_requirements.html
[^2]: https://docs.ansible.com/ansible/latest/reference_appendices/release_and_maintenance.html#ansible-core-support-matrix
[^3]: https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_structure.html#collection-structure
[^4]: https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html#magic-variables
[^5]: https://ansible.readthedocs.io/projects/lint/rules/
[^6]: https://yamllint.readthedocs.io/en/stable/rules.html
[^7]: https://github.com/ansible-collections/collection_template/tree/main
[^8]: https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_testing.html
[^9]: https://docs.ansible.com/ansible/latest/dev_guide/testing_running_locally.html#testing-running-locally
[^10]: https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_testing.html#testing-collections
[^11]: https://medium.com/@andrewjamesdawes/run-ansible-test-integration-tests-locally-on-docker-and-over-ssh-d8d658ba118b

[a]: https://goetzrieger.github.io/ansible-collections/5-creating-collections/
[b]: https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_structure.html#collections-doc-dir
[c]: https://github.com/ansible-collections/community.general/blob/main/tests/unit/plugins/filter/test_json_patch.py
[d]: https://github.com/ansible-collections/community.general/blob/main/tests/integration/targets/filter_dict/tasks/main.yml
[e]: https://yamllint.readthedocs.io/en/stable/
[f]: https://ansible.readthedocs.io/projects/lint/rules/
[g]: https://docs.ansible.com/ansible/devel/dev_guide/developing_collections_distributing.html#ignoring-files-and-folders
[h]: https://semver.org/#semantic-versioning-200
[j]: https://github.com/r-pufky/ansible_collection_srv
[k]: https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_creating.html