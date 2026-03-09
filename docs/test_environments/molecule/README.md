# Molecule
!!! info "Molecule 25.2+ [introduced breaking changes](../vagrant/troubleshooting.md#error-couldnt-resolve-moduleaction-vagrant)."


## Setup Framework

``` bash
# Directory may also be copied from other existing roles and updated.
# Molecule always uses current working directory.
cd roles/{ROLE}

# 'default' test using Podman driver.
molecule init scenario --driver-name=podman

# 'ssl' test using Vagrant driver.
molecule init scenario ssl --driver-name=vagrant
```

### [Directory layout][a]
These may be **yml** files or directories with **yml** files inside.

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


## Testing
Molecule deletes containers after every test cycle regardless of success or
failure. Use [best practices](../../best_practice/testing.md).

!!! tip "Use **--destroy=never** to retain containers for inspection"

``` bash
# Manual test steps.
molecule create
molecule converge -- -v
molecule verify

# Run through the default scenario or specified scenarios
molecule test  # Runs default test - always destroy test containers.
molecule test --destroy=never # Highly recommended. Keep test containers.

# scenario-name may be used in each step (create, convert, etc.).
molecule test --scenario-name=alt_test  # Runs 'alt_test'.

# Run through all existing Molecule scenarios
molecule test
molecule test --all
molecule test --all -- -v
molecule test --all --destroy=never
```

## Debug Molecule
Debug molecule actions, not the tests themselves.

``` bash
molecule --debug ${COMMAND}  # Enable verbose debugging.
```


[a]: https://sbarnea.com/slides/molecule/#/13
