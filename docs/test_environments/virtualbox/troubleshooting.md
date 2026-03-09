# Troubleshooting


## vboxdrv kernel module is not loaded
Kernel modules are not loaded.

!!! danger ""
    ``` log
    WARNING: The vboxdrv kernel module is not loaded. Either there is no module
             available for the current kernel (4.4.0-22-generic) or it failed to
             load. Please recompile the kernel module and install it by

               sudo /sbin/rcvboxdrv setup

             You will not be able to start VMs until this problem is fixed.
    ```

``` bash
# Install kernel modules.
pacman -Q | grep linux
pacman -S linux-{KERNEL}-virtualbox-host-modules
systemctl reboot

# Confirm installed.
vboxmanage --version  # Executable with modules loaded.
> 7.1.4r165100  # Installed version.
```
