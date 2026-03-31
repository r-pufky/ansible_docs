# Patterns

## Network Mounts
For mounted data locations (NFS with squashed mounts, mapped container mounts,
etc) the local root user will not have sufficient privileges to modify these
files.

Typically only the service user is mapped to the hosted UID/GID, and all other
users including root are mapped to 165536+ to prevent access (appears as
nobody/nogroup on opposing node).

Always attempt to first execute user data changes with the most common case
first and fallback to the service user if it fails.

``` yaml
- name: '{{ "Init DB | filesystem | set " ~ maria__init_db._data_d }}'
  block:
    - name: '{{ "Init DB | filesystem | set " ~ maria__init_db._data_d }}'
      ansible.builtin.file:
        path: '{{ maria__init_db._data_d }}'
        owner: '{{ maria__cfg._uid }}'
        group: '{{ maria__cfg._gid }}'
        mode: '0755'
        state: 'directory'
  rescue:
    - name: '{{ "Init DB | filesystem | set " ~ maria__init_db._data_d }}'
      ansible.builtin.file:
        path: '{{ maria__init_db._data_d }}'
        owner: '{{ maria__cfg._uid }}'
        group: '{{ maria__cfg._gid }}'
        mode: '0755'
        state: 'directory'
      become: true
      become_user: '{{ maria__srv._user }}'
```

## Templated Directories
Recursively templating directories separates application config format changes.

This enables the role to focus on critical setup and the consumer to configure
the application settings appropriately.

``` yaml
- name: '{{ "Install | filesystem | deploy " ~ maria___app._conf_d }}'
  ansible.builtin.template:
    src: '{{ item.src }}'
    dest: '{{ maria___app._conf_d }}'
    owner: '{{ maria__srv._user }}'
    group: '{{ maria__srv._group }}'
    mode: '0644'
  loop: '{{
      query("community.general.filetree", "templates/default/mariadb.conf.d")
    }}'
  loop_control:
    label: '{{ item.path }}'
  when: item.state == 'file'
  register: maria__local

- name: 'Install | filesystem | query remote files'
  ansible.builtin.find:
    paths: '{{ maria___app._conf_d }}'
    file_type: 'file'
    recurse: true
  register: maria__remote

- name: 'Install | filesystem | remove unmanaged files'
  ansible.builtin.file:
    path: '{{ item.path }}'
    state: 'absent'
  loop: '{{ maria__remote.files }}'
  loop_control:
    label: '{{ item.path }}'
  when: >
    item.path not in maria__local.results | selectattr('dest', 'defined') |
    map(attribute='dest') | list
```

!!! tip "Do not separate into another role"
    **community.general.filetree** resolves paths based on the play it was
    called from, not the path provided. This is different than standard ansible
    **include_tasks** behavior.

    This will break if this pattern is used outside of the play.

## Test remote filesystem permissions
Remote filesystem permission check causes stackdump for specific remote mounted
filesystems. Explicitly test remote file before applying file changes.

Settings permissions via octal **0755** vs symbolic **u=,g=,o=...** results in
plowing file permissions without checking first.

Remote filesystems typically do not have the same or allow the root user to
plow those file permissions (e.g. NFS squashed root or containers). Becoming
'root' locally results in the root user mapped to nobody/nogroup, effectively
locking the task out of modifying permissions even if NO change is required.

This results in ansible stack dumps that are hard to debug due to multiple
system layers on top of vague error messaging from ansible during simple file
permission setting (especially when pre-existing with correct permissions).

``` yaml
# Args:
#   file_path: str - Path to file.
#   file_owner: str or int - File owner.
#   file_group: str or int - File group.
#   file_mode: str - File mode.
#   file_recurse: bool - Enable recursive attribute setting.
#   file_state: str - File state.
#   file_diff: bool - Enable diff display. Default: False.
#
# Reference:
# * https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html

- name: 'Remote file | check {{ file_path }}'
  become: true
  ansible.builtin.file:
    path: '{{ file_path }}'
    owner: '{{ file_owner }}'
    group: '{{ file_group }}'
    mode: '{{ file_mode }}'
    state: '{{ file_state }}'
  check_mode: true
  diff: '{{ file_diff | default(false) }}'
  register: _utils_test

- name: 'Remote file | change {{ file_path }}'
  when: _utils_test.changed  # noqa no-handler execute immediately.
  become: true
  ansible.builtin.file:
    path: '{{ file_path }}'
    owner: '{{ file_owner }}'
    group: '{{ file_group }}'
    recurse: '{{ file_recurse }}'
    mode: '{{ file_mode }}'
    state: '{{ file_state }}'
  changed_when: not file_recurse  # https://github.com/ansible/ansible/issues/32636
```

## Atomically write INI files
Writes a given INI file containing duplicate keys atomically.

This is used in the case of INI files which contain duplicate keys (e.g. UE4
games). INI files that contain duplicate keys violate the INI standard. Files
are written to temp file with exclusive disabled, then moved into place,
preventing infinite duplicate key growth in configs.

This should only be used for INI files meeting this specific case; standard
cases should ALWAYS use **community.general.ini_file** directly in
configuration tasks.

``` yaml
# Args:
#   ini: list of dict - INI configuration.
#     - section: str - INI section heading to use.
#       option: str - INI section option.
#       value: any - INI section value (cast to string).
#       state: str - Add or remove specified key.
#           Values:
#             present: specified option=values lines are added, but other
#                      options with the same name are not touched.
#              absent: specified option=value lines are removed, but other
#                      options with the same name are not touched.
#       comment: str - Setting comment. Optional.
#
# Additional supported arguments mirror community.general.ini_file:
#   allow_no_value: bool - Allow option without value and without =.
#       Default: False.
#   attributes: str - Attributes resulting filesystem object should have in the
#         same order as the one displayed by lsattr. The = operator is assumed
#         as default, otherwise + or - operators need to be included in string.
#       Default: omit.
#   backup: bool - Create backup file with timestamp on change?
#       Default: False.
#   create: bool - Fail if file does not already exist.
#       Default: True.
#   follow: bool - Follow symlinks if they exist.
#       Default: False.
#   group: str - Group that owns filesystem object, per chown. Current group of
#         current user is used when unspecified, unless root in which case it
#         can preserve previous ownership.
#       Default: omit.
#   ignore_spaces: bool - Do not change line if doing so would only add or
#         remove spaces before or after =.
#       Default: False.
#   mode: str - Octal file permissions. Umask used if file created and mode not
#         specified, otherwise permissions are kept.
#       Default: omit.
#   modify_inactive_option: bool - Replace commented line that matches given
#         option.
#       Default: True.
#   no_extra_spaces: bool - Do not insert spaces before and after =.
#       Default: False.
#   owner: str - User name owning filesystem object, per chown. Current name of
#         current user is used when unspecified, unless root in which case it
#         can preserve previous ownership.
#       Default: omit.
#   path: str - Path to INI-style file; this file is created if required.
#   section_has_values: list of dict - With state=present, if a suitable
#         section is not found, a new section is added, including required
#         options. With state=absent, at most one section is removed if it
#         contains the values.
#       Default: 'present'.
#   selevel: str - SELinux filesystem object context level.
#       Default: omit.
#   serole: str - SELinux filesystem object context role.
#       Default: omit.
#   setype: str - SELinux filesystem object context type.
#       Default: omit.
#   seuser: str - SELinux filesystem object context user.
#       Default: omit.
#   unsafe_writes: bool - Use atomic operation to prevent data corruption or
#         inconsistent reads from the target filesystem object. This atomic
#         operation is different from atomic_ini (per write vs. per file).
#       Default: False.
#   check_mode: bool - Return status prediction without modifying target.
#       Default: False.
#   diff: bool - Return details on change.
#       Default: False.
#
# Reference:
# * https://docs.ansible.com/ansible/latest/collections/community/general/ini_file_module.html

- name: 'Atomic INI | stage'
  when: ini | length > 0
  community.general.ini_file:  # noqa name[template] filename
    allow_no_value: '{{ allow_no_value | default(false) }}'
    attributes: '{{ attributes | default(omit) }}'
    backup: '{{ backup | default(false) }}'
    create: '{{ create | default(true) }}'
    exclusive: false
    follow: '{{ follow | default(false) }}'
    group: '{{ group | default(omit) }}'
    ignore_spaces: '{{ ignore_spaces | default(false) }}'
    mode: '{{ mode | default(omit) }}'
    modify_inactive_option: '{{ modify_inactive_option | default(true) }}'
    no_extra_spaces: '{{ no_extra_spaces | default(false) }}'
    option: '{{ item.option }}'
    owner: '{{ owner | default(omit) }}'
    path: '{{ "/tmp/" ~ inventory_hostname ~ ".atomic_ini" }}'
    section: '{{ item.section }}'
    section_has_values: '{{ section_has_values | default(omit) }}'
    selevel: '{{ selevel | default(omit) }}'
    serole: '{{ serole | default(omit) }}'
    setype: '{{ setype | default(omit) }}'
    seuser: '{{ seuser | default(omit) }}'
    state: '{{ item.state | default("present") }}'
    unsafe_writes: '{{ unsafe_writes | default(false) }}'
    value: '{{ item.value | string }}'
  check_mode: '{{ check_mode | default(false) }}'
  diff: '{{ diff | default(false) }}'
  loop: '{{ ini }}'

- name: 'Atomic INI | set {{ path }}'
  when: ini | length > 0
  ansible.builtin.copy:
    src: '{{ "/tmp/" ~ inventory_hostname ~ ".atomic_ini" }}'
    remote_src: true
    dest: '{{ path }}'
    owner: '{{ owner | default(omit) }}'
    group: '{{ group | default(omit) }}'
    mode: '{{ mode | default(omit) }}'
    force: true
  check_mode: '{{ check_mode | default(false) }}'
  diff: '{{ diff | default(false) }}'

- name: 'Atomic INI | unstage'
  ansible.builtin.file:
    path: '{{ "/tmp/" ~ inventory_hostname ~ ".atomic_ini" }}'
    state: 'absent'
  check_mode: '{{ check_mode | default(false) }}'
  diff: '{{ diff | default(false) }}'
```
