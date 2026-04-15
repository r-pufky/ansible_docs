# Testing
Each pattern requires a separate molecule test setup.

!!! success "Use Podman"
    VMs are **ONLY** used to test specific cases which cannot be tested in
    containers (kernel, firmware, proc, systemd, networking, etc.) these should
    **never** be default test cases.

!!! success "All tests must pass"
    Use feature flags to require explicit enablement of tests which require
    user interaction to complete. Skip test and provide debug message when
    **not** enabled.

    **All** tests must pass with **molecule test --all**.

## Test Patterns
* Use **r_pufky.data.test** to handle a majority of common test cases with a
  single task.
* Do not re-test encapsulated roles and collections.
* Focus on testing all options and code paths. A component may test many
  options if they can be individually explicitly verified.

    !!! warning "Create [decisions](style_guide.md#file-documentation) when explicitly **NOT** testing a task."

* Use least privilege. Comments are required when using these features in
  Molecule:
    * Skipping **idempotent** step.
    * **root** or **SYS_** capabilities.
    * **gather_facts: true**.
* Molecule test name reflects components tested.
* Make use of [Molecule test variables][a].

## molecule.yml
* [Document Test](#documenting-tests).
* Define test variables for test setup.

## converge.yml
* **unsafe** variables [must be defined here][b].
* Edge cases outside of the base setup (e.g. multiple convergence steps).
* Use **mock** scripts to replace binaries when binaries cannot be used in a
  container and [they do **not** affect testing][c].

## prepare.yml
* Generate [static testing data (certs, keys, network)][d]

* Cache generated files in molecule directory (or use **r_pufky.data.test**):
    ``` yaml
    '{{ lookup("env", "MOLECULE_PROJECT_DIRECTORY") ~ "/molecule/cache" }}'
    ```
* Cache runtime test files in **/tmp** on remote host.
* Specify [dynamic target files][e].
* Load [generated files and inject in converge.yml][f].

## verify.yml
* Use **ansible.builtin.assert** to [validate test results][g]. Any tests missing
  assert are not explicitly validating correctness.

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
Effectively an role integration test with default values - only disabling
components which cannot be tested on testing platform - to validate no runtime
errors.

!!! abstract "converge.yml"
    0644 {USER}:{USER}

    ``` yaml
    - name: 'Converge | default role settings'
      ansible.builtin.include_role:
        name: 'r_pufky.deb.os'
    ```

### Regressions
Explicitly test major bugs to ensure bug is fixed.

* Prefix test with **regression_**.
* Reference bug in **verify.yml** with detailed information on cause and
  resolution.

## Assertions
Clearly state [expected results and failure reasons][h].

``` yaml
# Simple assertion failure
- name: 'Verify | script files | assert scripts'
  ansible.builtin.assert:
    quiet: true
    that:
      - test_nzbget_skel.files | length == 2
    fail_msg: '✘ /var/lib/nzbget/scripts{scan,post_processing} not found.'

# Complex error with variables
- name: 'Verify | expressions | assert nzbget_extensions'
  ansible.builtin.assert:
    quiet: true
    that:
      - test_nzbget_config is search('Extensions=scan,post_processing')
    fail_msg: |

      ✘ nzbget_extensions format incorrect.

      Expected:
        Extensions=scan,post_processing

       test_nzbget_config: {{ test_nzbget_config }}
```

[a]: https://ansible.readthedocs.io/projects/molecule/configuration/#molecule.config.Config
[b]: https://github.com/ansible/molecule/issues/4348
[c]: https://github.com/r-pufky/ansible_secure_boot/blob/main/molecule/default/converge.yml
[d]: https://github.com/r-pufky/ansible_sonarr/blob/main/molecule/enable_ssl/prepare.yml
[e]: https://github.com/r-pufky/ansible_ssh/blob/main/molecule/default/prepare.yml
[f]: https://github.com/r-pufky/ansible_ssh/blob/main/molecule/default/converge.yml
[g]: https://www.puppeteers.net/blog/ansible-quality-assurance-part-1-ansible-variable-validation-with-assert
[h]: https://github.com/r-pufky/ansible_nzbget/blob/main/molecule/variable_expressions/verify.yml