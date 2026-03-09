# Testing

!!! info "Prerequisites"
    * [Podman](../podman/README.md) - Primary test framework.
    * [Vagrant](../vagrant/README.md) - Secondary test framework.
    * [Molecule Setup](README.md).

## Execute Tests

Manual test steps
``` bash
molecule create
molecule converge -- -v
molecule verify
```
!!! tip ""
    `scenario-name` may be used in each step to execute non-default scenario.

Run through the default scenario or specified scenarios
``` bash
molecule test  # Runs default test - always destroy test containers.
molecule test --scenario-name=alt_test  # Runs 'alt_test'.
```

Run through all existing Molecule scenarios
``` bash
molecule test
molecule test --all
molecule test --all -- -v
molecule test --all --destroy=never  # Run all tests - keep containers.
```

## Debug molecule state
``` bash
molecule --debug ${COMMAND}  # Enable verbose debugging.
```
