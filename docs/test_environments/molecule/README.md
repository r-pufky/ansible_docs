# Molecule Setup

!!! info "Prerequisites"
    * [Podman](../podman/README.md) - Primary test framework.
    * [Vagrant](../vagrant/README.md) - Secondary test framework.

## Create Test
Directory may also be copied from other existing roles and updated.

``` bash
cd roles/{ROLE}  # molecule always uses current working directory.

# 'default' test using Podman driver.
molecule init scenario --driver-name=podman

# 'ssl' test using Vagrant driver.
molecule init scenario ssl --driver-name=vagrant
```

## Directory layout
These may be `yml` files or directories with `yml` files inside.

``` yaml
{ROLE}
├── dependency
├── lint
├── cleanup
├── destroy
├── syntax
├── create
├── prepare  # Bring host to testable status.
├── converge  # Required - apply role to test.
├── idempotence
├── side_effect
├── verify  # Assert role is correct.
├── cleanup
╰── destroy
```
Reference:

* https://ansible.readthedocs.io/projects/molecule/getting-started/#inspecting-the-moleculeyml
* https://sbarnea.com/slides/molecule/#/13
