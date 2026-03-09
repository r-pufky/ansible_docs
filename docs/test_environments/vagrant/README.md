# Vagrant Setup

!!! info "Prerequisites"
    * [VirtualBox](../virtualbox/README.md).

!!! warning "Explicit Need Only"
    Vagrant VMs are **ONLY** used to test cases which cannot be tested
    in containers (kernel, firmware, proc, systemd, networking, etc) these
    should **never** be default test cases.

## Install

``` bash
source {VENV}/bin/activate
pamac install vagrant
pip install molecule-plugins[Vagrant]
```
Reference:

* https://wiki.archlinux.org/title/Vagrant

## Set alternative storage location (optional)
Developing on vagrant will thrash disk especially when running molecule with
large (multi-GB) reads and writes. Relocate high-use directories to a disk that
can handle high wear. Prefer to config change as this enables quick use without
configuration changes.

Move and link user directories to an alternative location (1)
{ .annotate }

1. Consider moving and linking entire `.cache` directory.

``` bash
# delete or move existing cache data.
ls -s /mnt/cache/local/share/containers ${HOME}/.local/share/containers
```
