# Vagrant

!!! warning "Explicit Need Only"
    Vagrant managed VMs are **ONLY** used to test specific cases which cannot
    be tested in containers (kernel, firmware, proc, systemd, networking, etc.)
    these should **never** be default test cases.


## System Images Used

* [debian/trixie64][a]
* [inception-of-things/trixie][b]
* Alternatively a [non-maintained Debian image may be created.][c].

Image repositories:

* https://portal.cloud.hashicorp.com/vagrant/discover
* https://www.osboxes.org/debian

See [libvirt configuration with vagrant](../libvirt/README.md).


## Reference[^1][^2][^3]

[^1]: https://wiki.archlinux.org/title/Vagrant
[^2]: https://ansible.readthedocs.io/projects/molecule/configuration/?h=test_sequence#scenario
[^3]: https://floatingpoint.sorint.it/blog/post/setting-up-molecule-for-testing-ansible-roles-with-vagrant-and-testinfra


[a]: https://portal.cloud.hashicorp.com/vagrant/discover/debian/trixie64
[b]: https://portal.cloud.hashicorp.com/vagrant/discover/inception-of-things/debian-trixie
[c]: https://raju.dev/building-debian-13-trixie-vagrant-image
