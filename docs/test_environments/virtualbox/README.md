# VirtualBox Setup

!!! warning "Explicit Need Only"
    VirtualBox VMs are **ONLY** used to test cases which cannot be tested
    in containers (kernel, firmware, proc, systemd, networking, etc) these
    should **never** be default test cases.

## Install

``` bash
source {VENV}/bin/activate
pamac install virtualbox  # No additional dependencies.
mhwd-kernel -l
mhwd-kernel -li  # Install at least the running kernel.
pamac install linux{VERSION}-virtualbox-host-modules
pip install molecule-plugins[VirtualBox]
gpasswd -a ${USER} vboxusers  # Add user to virtualbox users.
systemctl reboot  # Install extended features first if needed.
```
Reference:

* https://wiki.manjaro.org/index.php/VirtualBox

Extended features are only needed for testing USB, webcam, RDP, PXE, disk
encryption, or NVMe.
``` bash
pamac build virtualbox-ext-oracle
systemctl reboot

modprobe vboxdrv vboxnetadp xvboxnetflt  # Alternate reload without reboot.
```

Confirm installed.
``` bash
vboxmanage --version
> 7.1.4r165100  # Installed version.
```

## Set alternative storage location (optional)
Developing on VMs will thrash disk especially when running molecule. Relocate
high-use directories to a disk that can handle high wear. Prefer to config
change as this enables quick use without configuration changes.

Move and link user directories to an alternative location (1)
{ .annotate }

1. Consider moving and linking entire `.cache` directory.

``` bash
# delete or move existing cache data.
ls -s /mnt/cache/config/VirtualBox ${HOME}/.config/VirtualBox  # Config.
ln -s '/mnt/cache/VirtualBox VMs' '${HOME}/VirtualBox VMs'  # VM data.
```
Reference:

* https://docs.oracle.com/en/virtualization/virtualbox/6.0/admin/vboxconfigdata.html
