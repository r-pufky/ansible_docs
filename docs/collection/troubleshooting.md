# Troubleshooting

## Testing in roles uses old files
Collection cache is outdated.

Clear build cache.
``` bash
rm -rfv ~/.ansible/collections/ansible_collections/r_pufky/srv
```

## Unable to determine context for the following test targets
A controller and a target were not specified when running the test.

??? danger "Error"
    ``` log
    WARNING: Unable to determine context for the following test targets, they will
    be run on the target host: {MODULE}, {MODULE}, they will be run on the target
    host: {MODULE}, {MODULE}
    ```

This is OK for tests that are run on localhost with NO impact on system (e.g.
filters).

## Only Localhost is Available
Host inventory not detected when running collection tests.

??? danger "Error"
    ``` bash
    ansible-playbook roles/hello_motd/tests/playbook/test_hello_motd.yml
    ...
      [WARNING]: No inventory was parsed, only implicit localhost is available
      [WARNING]: provided hosts list is empty, only localhost is available. Note
      that the implicit localhost does not match 'all'
    ```

Explicitly specify inventory to use.
``` bash
ansible-playbook -i tests/inventory tests/playbook/test_hello_motd.yml
```

!!! tip
    **--ask-become-pass** will be needed if sudo used or use GPG key injection.

## Fixing GPLv3 License Missing
Community distributions assumes GPLv3.

Explicitly ignore if different license.
``` bash
mkdir -p tests/sanity
touch tests/sanity/ignore-{ANSIBLE VERSION}.txt

touch tests/sanity/ignore-2.16.txt
```
!!! warning
    An ignore file is required for each specific **version** of ansible.

!!! tip
    **Order Matters**: if ignores are throwing warnings, check ordering and make
    sure they are ordered correctly. Proper ignores will **not** generate warnings.

!!! info
    Only certain ignores are explicitly allowed; others will always throw
    ansible-lint errors even if ignored.

tests/sanity/ignore-2.16.txt
```
plugins/modules/demo_hello.py compile-2.7!skip
plugins/modules/demo_hello.py import-2.7!skip
plugins/modules/demo_hello.py compile-3.6!skip
plugins/modules/demo_hello.py import-3.6!skip
plugins/modules/demo_hello.py compile-3.7!skip
plugins/modules/demo_hello.py import-3.7!skip
plugins/modules/demo_hello.py compile-3.8!skip
plugins/modules/demo_hello.py import-3.8!skip
plugins/modules/demo_hello.py compile-3.9!skip
plugins/modules/demo_hello.py import-3.9!skip
plugins/modules/demo_hello.py compile-3.10!skip
plugins/modules/demo_hello.py import-3.10!skip
plugins/modules/demo_hello.py compile-3.12!skip
plugins/modules/demo_hello.py import-3.12!skip
plugins/modules/demo_hello.py validate-modules:missing-gplv3-license
```
Reference:

* https://github.com/ansible/ansible/issues/67032
* https://docs.ansible.com/ansible/latest/dev_guide/testing/sanity/ignores.html
* https://docs.ansible.com/ansible/latest/dev_guide/testing/sanity/validate-modules.html
* https://ansible.readthedocs.io/projects/lint/rules/sanity/
