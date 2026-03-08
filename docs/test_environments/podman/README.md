# Podman Setup

!!! info "Prerequisites"
    * [Ansible Environment](../../ansible/environment/README.md).

!!! success "Default Testing Platform"
    Rootless podman environment is used to test **all** cases unless there are
    required bare-metal cases (See [Vagrant](../vagrant/README.md)).

## Install

=== "Arch"

    ``` bash
    pacman -Syu crun  # OCI implementation (faster, less memory than runc).
    pacman -Syu podman  # Service tests (non kernel, sysctl, networking, etc).
    ```

=== "Debian"

    ``` bash
    apt install crun  # OCI implementation (faster, less memory than runc).
    apt install podman  # Service tests (non kernel, sysctl, networking, etc).
    ```

``` bash
# source ansible.env if not using direnv.
source /var/venv/ansible/bin/activate
pip install molecule-plugins[Podman]
```

Verify Rootless Support. (1)
{ .annotate }

1. **overlay** and **Diff: "true"** mean supported.

``` bash
podman info | grep -i overlay

> 107:  graphDriverName: overlay
> 114:    Native Overlay Diff: "true"
```

Verify Unprivileged User Namespace Enabled. (1)
{ .annotate }

1. **1** means enabled.

``` bash
sysctl kernel.unprivileged_userns_clone

> kernel.unprivileged_userns_clone = 1
```

Create Subordinate UID/GID Mappings. (1)
{ .annotate }

1. Configuration entry must exist for each user that wants to use it. New users
   created using useradd have these entries by default. If not add user
   defaults.

``` bash
# Add user
cat /etc/subuid | grep {USER}
> {USER}:100000:65536

# Add group
cat /etc/subgid | grep {GROUP}
> {GROUP}:100000:65536

# Modern linux distros may use this
usermod --add-subuids 100000-165535 --add-subgids 100000-165535 {USER}
```
Reference:

* https://github.com/ansible-community/molecule-podman
* https://github.com/systemd/systemd/issues/21952

## Set Default Container Registry

/etc/containers/registries.conf (1)
{ .annotate }

1. 0644 root:root

``` ini
[registries.search]
registries = ['ghcr.io']
unqualified-search-registries=['ghcr.io']
```

``` bash
podman login ghcr.io  # github account.
```
!!! warning "Required"
    GHCR.IO requires a github login to download images.

Reference:

* https://halukkarakaya.medium.com/how-to-configure-default-search-registries-in-podman-ea930289692

## Migrate Podman Installation
Applies configuration changes to Podman. Critical for unprivileged podman to
execute properly.

Run as the **current** user.
``` bash
podman system reset
podman system migrate
```
Reference:

* https://github.com/containers/podman/issues/12715


## Set alternative container storage location (optional)
Developing on containers will thrash disk especially when running molecule.
Relocate high-use directories to a disk that can handle high wear. Prefer to
config change as this enables quick use without configuration changes.

!!! warning "graphroot and runroot are ignored in rootless containers"
    `graphroot` and `runroot` are **ignored** in rootless containers and use
    following defaults if not defined:
    ``` bash
    XDG_CONFIG_HOME=${HOME}/.config
    XDG_DATA_DIR=${HOME}/.local/share
    XDG_RUNTIME_DIR=/run/user/${UID}
    ```

Move and link user directories to an alternative location. (1)
{ .annotate }

1. Consider moving and linking entire **.cache** directory.

``` bash
# delete or move existing cache data.
# rootless relocation
ls -s /mnt/cache/local/share/containers ${HOME}/.local/share/containers  # graph.
ln -s /mnt/cache/cache/containers ${HOME}/.cache/containers  # config.
# podman relocation
ln -s /hdd/cache/storage /var/lib/containers/storage  # graph.
```
Reference:

* https://github.com/containers/podman/blob/main/docs/tutorials/rootless_tutorial.md#user-configuration-files

## References

* https://www.ansible.com/blog/developing-and-testing-ansible-roles-with-molecule-and-podman-part-1/
* https://wiki.archlinux.org/title/Podman
