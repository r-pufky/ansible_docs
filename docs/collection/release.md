# Major Release Guide

4. Update known settings to new version
    * Test images
        * `debian-systemd:12` ➔ `debian-systemd:13`
        * `debian/bookworm64` ➔ `inception-of-things/debian-trixie`
        * `debian-12-` ➔ `debian-`
    * Entire role
      ``` bash
      grep bookworm # Replace with trixie.
      grep 12.x # Replace with 13.x.
      grep fail:  # Update formatting.
      grep debug:  # Update formatting.
      grep msg:  # Update formatting.
      grep -ri todo  # Resolve any open TODO's.
      grep -ri noqa  # Minimize NOQA's for accepted ansible-lint disables.
      ```
    * Update **vars/main.yml** (dates, packages, etc.).
    * Fix any code warts.
    * Update docstring formats.
    * Implement MAJOR role changes.
    * Check updated packages for new variables.
    * Diff links (esp distro versions) for added/removed items, updated links.
      ``` bash
      meld \
        <(curl -s https://forgejo.org/docs/v11.0/admin/config-cheat-sheet/)
        <(curl -s https://forgejo.org/docs/v10.0/admin/config-cheat-sheet/)
      ```
    * Update **meta/main.yml**.

        !!! tip "Decision: Use aggressive deprecation"
            Remove previous versions when in-depth workarounds are required.
            Previous OS support encoded in historical build artifacts.

    * Update **galaxy.yml**.
    * Update **changelogs/changelog.yaml**

        Truncate changelog and use `ancestor: {LAST VERSION}` to link to
        previous version changelog.

## Versioning
Default to schematic versioning - each signifier may inherit versioning scheme
used for underlying system.

## Build
``` bash
ansible-galaxy collection build
```
Sanity check size/contents for unwanted inclusions.

## Release

1. Upload new artifact to https://galaxy.ansible.com

    Confirm that collection markdown is formatted correctly (Ansible uses a
    subset of GFM to render documents and may differ from Github). Rebuild and
    re-upload as needed.

2. Create github release.

    * Use **galaxy.yml** for commit template. See previous releases.

## Release Order
Collections depend on lower level dependencies. Release in this order:

1. data
2. lib
3. deb
4. srv
5. arr
6. game
7. docs
