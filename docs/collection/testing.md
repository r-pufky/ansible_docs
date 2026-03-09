# Testing
Use community references for examples on specific implementations:

* [Filter Plugins](https://github.com/ansible-collections/community.general/blob/main/tests/unit/plugins/filter/test_json_patch.py)
* [Integration Tests](https://github.com/ansible-collections/community.general/blob/main/tests/integration/targets/filter_dict/tasks/main.yml)

## Build and Install Local Collection
The collection must be built and installed locally to be tested.

Package built using semantic version from `galaxy.yml` and installed to local
collections cache:
`~/.ansible/collections/ansible_collections/{USER}/{COLLECTION}`.

``` bash
ansible-galaxy collection build -f  # -f (force) useful for repeated testing.
ansible-galaxy collection install -f  # -f (force) useful for repeated testing.
```
Collection **will** need to be rebuilt if changes (files or tests) were made.

## Running Tests
**Any change** requires rebuilding, installing **AND** re-entering directory as
inodes change.

Alway [build and install](#build-and-install-local-collection) before running
tests.

``` bash
# Always re-enter between installs - cd ~; cd -
cd ~/.ansible/collections/ansible_collections/{USER}/{COLLECTION}
ansible-test {unit,integration,sanity}
```
Reference:

* https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_testing.html
* https://docs.ansible.com/ansible/latest/dev_guide/testing_running_locally.html#testing-running-locally
* https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_testing.html#testing-collections
* https://medium.com/@andrewjamesdawes/run-ansible-test-integration-tests-locally-on-docker-and-over-ssh-d8d658ba118b
