# VirtualBox Setup

!!! bug "Not migrated to environment configuration"
    Documentation format updated but not migrated to main environment section
    as very likely VirtualBox will be removed in favor of libvirt going
    forward.

!!! warning "Explicit Need Only"
    VirtualBox VMs are **ONLY** used to test specific cases which cannot be
    tested in containers (kernel, firmware, proc, systemd, networking, etc.)
    these should **never** be default test cases.


## [Install][a]

``` bash
source {VENV}/bin/activate
pacman -S virtualbox  # No additional dependencies.
pacman -Q | grep linux
pacman -S linux-{KERNEL}-virtualbox-host-modules
pip install molecule-plugins[virtualbox]
gpasswd -a ${USER} vboxusers  # Add user to virtualbox users.
systemctl reboot  # Install extended features first if needed.

# Extended features are only needed for testing USB, webcam, RDP, PXE, disk
# encryption, or NVMe.
paru -S virtualbox-ext-oracle
systemctl reboot

modprobe vboxdrv vboxnetadp xvboxnetflt  # Alternate reload without reboot.
```

Confirm installed.
``` bash
vboxmanage --version  # Executable with modules loaded.
> 7.1.4r165100  # Installed version.
```


## Set [alternative storage location][b]
Developing on VMs will thrash disk especially when running molecule. Relocate
high-use directories to a disk that can handle high wear. Prefer to config
change as this enables quick use without configuration changes.

``` bash
# Consider moving and linking entire `.cache` directory.
# delete or move existing cache data.
ls -s /mnt/cache/config/VirtualBox ${HOME}/.config/VirtualBox  # Config.
ln -s '/mnt/cache/VirtualBox VMs' '${HOME}/VirtualBox VMs'  # VM data.
```

[a]: https://wiki.manjaro.org/index.php/VirtualBox
[b]: https://docs.oracle.com/en/virtualization/virtualbox/6.0/admin/vboxconfigdata.html