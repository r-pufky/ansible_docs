# Best Practices
Each pattern requires a separate molecule test setup.

## Test Patterns
* Do **NOT** re-test encapsulated roles and collections.
* Focus on testing **all** options and **code paths**. A component **may** test
  many options if they can be individually explicitly verified.

    !!! warning "Use Decisions"
        Create [decisions](../../best_practice/style_guide.md#file-documentation)
        when explicitly **NOT** testing a task.

* Use least privilege. Comments are required when using these features in
  Molecule.
    * Skipping `idempotent` step.
    * `root` or `SYS_` capabilities.
    * `gather_facts: true`.
* Roles should be thoroughly tested with VMs for tasks not covered in
  containers.
* Molecule test name reflects component tested.
* **All** tests must pass with `molecule test --all`.

    !!! warning "Require Enablement"
        Live testing (e.g. external API use) requires **explicit** enablement
        (See [example](https://github.com/r-pufky/ansible_repo/blob/main/molecule/live_api_test/converge.yml)).

        Live testing should **always** occur before submitting changes if code
        paths were affected.

* Make use of [Molecule test variables](https://ansible.readthedocs.io/projects/molecule/configuration/#molecule.config.Config)

## molecule.yml
* Describe test.
* Define test variables for test setup.

## converge.yml
* **unsafe** variables **must** be defined here.
  [Reference](https://github.com/ansible/molecule/issues/4348).
* Edge cases outside of the base setup (e.g.
  multiple convergence steps).
* Use **mock** scripts to replace binaries when binaries cannot be used in a
  container and they do **not** affect testing (See [example](https://github.com/r-pufky/ansible_secure_boot/blob/main/molecule/default/converge.yml)).
* Do **live** testing here (See [example](https://github.com/r-pufky/ansible_repo/blob/main/molecule/live_api_test/converge.yml)).
    * Use `{ROLE}_testing_enable`.
    * Disable live API testing by default to ensure all tests pass.
    * Disable live API usage (which cannot be mocked easily).
## prepare.yml
* Generate static testing data (certs, keys, network) (See
  [example](https://github.com/r-pufky/ansible_sonarr/blob/main/molecule/enable_ssl/prepare.yml)).
* Cache generated files in molecule directory:
    ``` yaml
    '{{ lookup("env", "MOLECULE_PROJECT_DIRECTORY") ~ "/molecule/cache" }}'
    ```
* Cache runtime test files in **/tmp** on remote host.
* Specify dynamic target files (See
  [example](https://github.com/r-pufky/ansible_ssh/blob/main/molecule/default/prepare.yml)).
* Load generated files and inject in `converge.yml` (See
  [example](https://github.com/r-pufky/ansible_ssh/blob/main/molecule/default/converge.yml)).

## verify.yml
* Use `ansible.builtin.assert` to validate test results. Any tests missing
  assert are not explicitly validating correctness.

Reference:

* https://www.puppeteers.net/blog/ansible-quality-assurance-part-1-ansible-variable-validation-with-assert/

## Documenting Tests
Explicitly state test conditions in `molecule.yml` using the following format:

``` yaml
###############################################################################
# [Molecule Test Name]
###############################################################################
# Description of the molecule test / setup.
#
# Tests:
# * Each condition to verify on completion.
```

### Default
`default` is effectively an role integration test with default values - only
disabling components which cannot be tested on testing platform - to validate
no runtime errors.

converge.yml
``` yaml
- name: 'Converge | default role settings'
  ansible.builtin.include_role:
    name: 'r_pufky.deb.os'
```

### Regression
Explicitly test major bugs to ensure bug is fixed.

* Prefix test with **regression_**.
* Reference bug in `verify.yml` with detailed information on cause and
  resolution.

## Assertions
Clearly state expected results and failure reasons (See [example](https://github.com/r-pufky/ansible_nzbget/blob/main/molecule/variable_expressions/verify.yml)).

### Simple assertion failure
``` yaml
- name: 'Verify | script files | assert scripts'
  ansible.builtin.assert:
    quiet: true
    that:
      - _test_nzbget_skel.files | length == 2
    fail_msg: |
      ╭───────────────────────────────────────────────────────────────────╮
      │                                                                   │
      │     /var/lib/nzbget/scripts{scan,post_processing} not found.      │
      │                                                                   │
      ╰───────────────────────────────────────────────────────────────────╯
```

### Complex error with variables
``` yaml
- name: 'Verify | expressions | assert nzbget_extensions'
  ansible.builtin.assert:
    quiet: true
    that:
      - _test_nzbget_config is search('Extensions=scan,post_processing')
    fail_msg: |
      ╭───────────────────────────────────────────────────────────────────╮
      │                                                                   │
      │                nzbget_extensions format incorrect.                │
      │                                                                   │
      ├───────────────────────────────────────────────────────────────────╯
      │ Expected:
      │   Extensions=scan,post_processing
      │
      │ _test_nzbget_config: {{ _test_nzbget_config }}
```
