# Style Guide

!!! tip
    Prefer readability and consistency across files minimizing exceptions.

!!! warning
    **Always** follow **yamllint** and **ansible-lint** linting with **no**
    global exceptions. See
    [examples](https://github.com/r-pufky/ansible_paperless_ngx).

## 80 character width
Only extend past 80 characters if long variable names make it impossible.

```yaml
- ansible.builtin.set_fact:
    JWT_SIGNING_PRIVATE_KEY_FILE:
      '{{ forgejo_config_server_app_data_path }}/jwt/private.pem',
    JWT_SIGNING_ALGORITHM: '{{
        forgejo_config_oauth2_jwt_signing_algorithm | upper
      }}',
    SSH_TRUSTED_USER_CA_KEYS_FILENAME: '{{
        _forgejo_home ~ "/.ssh/ca_trust_chain_root.pem"
        if forgejo_config_server_ssh_trusted_user_ca_keys_filename | length > 0 else
        ""
      }}',

- ansible.builtin.set_fact:
    _unifi_cfg_legacy_properties_unifi_https_ssl_enabled_protocols: "{{
      {} | r_pufky.data.v3(
        section='system.properties',
        key='unifi.https.sslEnabledProtocols',
        raw=unifi_cfg_legacy_properties_unifi_https_ssl_enabled_protocols,
        data=(
          unifi_cfg_legacy_properties_unifi_https_ssl_enabled_protocols |
          join(',')
        ),
        default=['TLSv1', 'SSLv2Hello'],
        hint='list of str',
        keep=false,
        order=17
        )
      }}"
```

* Single-line uses **2** space block (prefer).
* Multi-line vars use hanging opens with **4** space block and **2** closer;
  lines may go over 80 characters if it cannot be split further using these
  rules.
* If statements use hanging opens with **4** space block and **2** closer; one
  line per operator and may each individually go over 80 character limit.

## Accepted Disables

### line-length
``` yaml
---
# yamllint disable rule:line-length
...
# yamllint enable rule:line-length
some_long_yaml_line: 0  # yamllint disable-line rule:line-length
```

* Must appear immediately after YAML specifier if file disabled.
* Enable as soon as possible if file level disable.

### jinja[spacing]
Disable for complex tasks (such as filters or combining dicts) - prefer human
readability.

``` yaml
- name: 'Annotate | sanitize & annotate service defaults'
  ansible.builtin.set_fact:  # noqa jinja[spacing] readability.
    _unifi_srv_legacy_enable: "{{ {} | r_pufky.data.v3(
        raw=unifi_srv_legacy_enable,
        default=true,
        _dir='/var/lib/unifi',
        _properties='/var/lib/unifi/system.properties',
        hint='bool',
        order=1
        )
      }}"
```

### name[template]
For multiple variables in a name directive - typically file paths.

``` yaml
- name: 'file: {{ item.path }}/.hidden_file'  # noqa name[template] file path
```

### no-handler
Immediately execute command or handler in cases where results are required for
role to progress.

``` yaml
- name: Update checksum'
  when: _test_prepare.changed  # noqa no-handler execute immediately.
  ansible.builtin.command:
    argv:
      - 'sed'
      - '-i'
      - 's/11.0.3/11.0.4/g'
      - '{{ _test_cache }}/forgejo-11.0.4-linux-amd64.sha256'
  changed_when: true
```

## Name Directives

### Tasks
Use bare descriptions with pipes for multi-part steps.

``` yaml
- name: 'Config'
  ansible.builtin.include_tasks: 'config.yml'  # tasks/main.yml

- name: 'File | disable firewall'  # tasks/file.yml
  when: not ufw_enable
  community.general.ufw:
    state: 'disabled'

- name: 'Sub section | file | disable firewall'  # tasks/sub_section/file.yml.
  when: not ufw_enable
  community.general.ufw:
    state: 'disabled'
```

### Clause Order
Prioritized for readability and clarity.


``` yaml
- name: 'task with a clause'
  when: some_var_to_test  # place 'when' next to loop if clause uses loop item.
  notify: 'handler'
  community.general.ufw:
    state: 'disabled'
  become: true
  register: _some_var
  loop: '{{ user_accounts }}'
  loop_control:
    label: '{{ item.user }}'
```

### Handler (listen clauses)
Use **listen** clause to run multiple handlers for a given task. **listen** can
be a list, allowing for specific combinations as needed.

Execution order is **always** the order as written in `handlers.yml`,
regardless of call order.

Handlers.
```yaml
- name: 'Handlers | ifreload'
  listen:
    - 'Handlers | reload restart networking'
    - 'Handlers | reload restart FRR'
  ansible.builtin.command: '/usr/sbin/ifreload -a'
  become: true
  changed_when: false

- name: 'Handlers | restart networking'
  listen: 'Handlers | reload restart networking'
  when: network_systemctl_network_restart
  ansible.builtin.service:
    name: 'networking'
    state: 'restarted'
  changed_when: true

- name: 'Handlers | restart FRR'
  listen: 'Handlers | reload restart frr'
  ansible.builtin.service:
    name: 'frr'
    state: 'restarted'
```

Usage.
``` yaml
- name: 'run restart networking'
  notify: 'Handlers | reload restart networking'
  ...

- name: 'run ifreload, restart networking'
  notify: 'Handlers | reload restart networking'
  ...

- name: 'run ifreload, restart FRR'
  notify: 'Handlers | reload restart FRR'
  ...
```

Reference:

* https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_handlers.html
* https://github.com/ansible/ansible/issues/16378

## File / Section Headers
One vertical space to allow for individual task and variable comments.

``` yaml
###############################################################################
# Capped Header Text (Clarifications)
###############################################################################
# Reference:
# * https://some/reference

- name: 'task with a clause'
  when: some_var_to_test
  community.general.ufw:
    state: 'disabled'
```
