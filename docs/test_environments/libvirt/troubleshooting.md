# Troubleshooting
!!! info "Molecule 25.2+ [introduced breaking changes](../vagrant/troubleshooting.md#error-couldnt-resolve-moduleaction-vagrant)."


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
