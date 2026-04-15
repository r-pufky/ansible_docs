# Troubleshooting

!!! info ""
    See [molecule](../molecule/troubleshooting.md) for specific molecule
    troubleshooting.

## Driver podman does not [provide a schema][a]
Podman driver does have a schema and will always generate a warning.

!!! danger ""
    ``` log
    WARNING Driver podman does not provide a schema.
    ```

## Podman [requires 'tmpfs' as dict][b] not list
[Molecule documentation][c] refers to using **tmpfs** as a list. Podman
**only** accepts dictionaries.

molecule.yml
``` yaml
platforms:
  - name: 'tmpfs as a dict'
    image: ghcr.io/hifis-net/debian-systemd:13
    tmpfs:
      /tmp: 'rw,size=787448k,mode=1777'

platforms:
  - name: 'tmpfs not needed with systemd always'
    image: 'ghcr.io/hifis-net/debian-systemd:13'
    systemd: 'always'  # tmpfs not need if 'always'.
```

## Podman database [static dir mis-match][d]
Podman will not migrate things out of **/home** to prevent existing pod
breakage.

!!! danger ""
    ``` log
    Error: database static dir ".../graph/libpod" does not match our static dir ".../graph/libpod": database configuration mismatch
    ```

``` bash
# Reset podman and migrate.
podman system reset
podman system migrate
```

## [Cannot use ping][e] from container
Some containers services require ping.

=== "Systemwide"

    !!! abstract "/etc/sysctl.d/net_ipv4_ping_group_range"
        0755 root:root

        ``` bash
        net.ipv4.ping_group_range=0 $MAX_UID
        ```

=== "Exection Only"

    !!! abstract "/etc/containers/containers.conf"
        0644 root:root

        ``` ini
        default_sysctls = [
          "net.ipv4.ping_group_range=0 2147483647",
        ]
        ```

## Containers [ignore graphroot and runroot][f]
Graphroot and runroot are ignored in rootless containers and use following
defaults if not defined:

``` bash
XDG_CONFIG_HOME=${HOME}/.config
XDG_DATA_DIR=${HOME}/.local/share
XDG_RUNTIME_DIR=/run/user/${UID}
```

[a]: https://github.com/ansible/molecule/discussions/4108
[b]: https://docs.ansible.com/ansible/latest/collections/containers/podman/podman_container_module.html#parameter-tmpfs
[c]: https://github.com/ansible/molecule/issues/4140
[d]: https://github.com/containers/podman/pull/20874
[e]: https://github.com/containers/podman/blob/main/troubleshooting.md#5-rootless-containers-cannot-ping-hosts
[f]: https://github.com/containers/podman/blob/main/docs/tutorials/rootless_tutorial.md#user-configuration-files