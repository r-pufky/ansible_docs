# Ansible Environment

!!! tip "Decision: Use isolated minimal environments"
    Each collection uses an isolated minimal environment to minimize hidden
    dependencies.

## Create Environment and Install Packages

=== "Arch"

    ``` bash
    pacman -S python-pip
    pacman -S direnv
    ```

=== "Debian"

    ``` bash
    apt install python3-pip python3-venv
    apt install direnv
    ```

``` bash
# Create python virtual environment.
python3 -m venv /var/venv/ansible  # tip: a220 for ansible-core 2.20
source /var/venv/ansible/bin/activate  # activate.fish

# Install and lock to specific version.
python -m ensurepip --upgrade
pip install --upgrade pip
pip install --upgrade setuptools
pip install ansible  # ansible==11.0  # ansible-core 2.20
pip install argcomplete
pip install ansible-lint
pip install molecule

direnv allow  # source ansible.env if not using direnv.
```
Reference:

* https://docs.ansible.com/ansible/latest/roadmap/ansible_roadmap_index.html
* https://docs.ansible.com/ansible/2.9/installation_guide/intro_installation.html

## Set alternative **.ansible** cache location
A recent change in Ansible now creates an **.ansible** directory in each
directory ansible is run - building and copying all collections/roles
recursively - potentially creating multi-GB PER directory. Leads to extremely
slow and noisy **yamllint** and **ansible-lint** usage.

/etc/fstab
``` bash
tmpfs  /tmp/ansible-cache  tmpfs  noatime,mode=1777  0 0
```

Mount new directory
``` bash
mount -o remount -a
```

Symlink all directories to cache
``` bash
find . -type d -name '.ansible' -exec rm -rf "{}" \; -exec ln -s /tmp/ansible-cache "{}" \;
```
Reference:

* https://github.com/ansible/ansible-lint/issues/4533

## Set alternative container storage location (optional)
Developing ansible will thrash disk especially when running playbooks and
testing with molecule. Relocate high-use directories to a disk that can handle
high wear. Prefer to config change as this enables vanilla use in production.

Link ansible caching directories. (1)
{ .annotate }

1. Consider moving and linking entire **.cache** directory.

``` bash
# Delete or move existing cache data.
ln -s /mnt/cache/.ansible_async ${HOME}/.ansible_async
ln -s /mnt/cache/.ansible ${HOME}/.ansible
ln -s /mnt/cache/cache/ansible-compat ${HOME}/.cache/ansible-compat
ln -s /mnt/cache/cache/molecule ${HOME}/.cache/molecule
```
