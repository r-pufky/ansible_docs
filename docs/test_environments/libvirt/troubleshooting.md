# Troubleshooting
!!! info "Molecule 25.2+ [introduced breaking changes](../vagrant/troubleshooting.md#error-couldnt-resolve-moduleaction-vagrant)."


## [dist-upgrade fails for grub-pc][b]
Virtual disk device mapping changed during configuration, requiring interactive
re-configuration. Re-create the VM.

!!! abstract ""

    ``` bash
    Errors were encountered while processing:
     grub-pc

    MSG:
    '/usr/bin/apt-get dist-upgrade ' failed: E: Sub-process /usr/bin/dpkg returned an error code (1)
    ```


## [No Network Connection][a]
libvirt switched to using nftables userspace directly. Immediate workaround is
to set libvirt back to iptables backend or disable UFW.

UFW uses legacy iptables userspace tools and these create shared nftables top
level tables. All different applications which use iptables userspace end up in
same nftables table.

!!! abstract "/etc/libvirt/network.conf"
    0644 root:root

    ``` ini
    firewall_backend = "iptables"
    ```


[a]: https://gitlab.com/libvirt/libvirt/-/work_items/644
[b]: https://stackoverflow.com/questions/44621077/vagrant-stalling-on-boot