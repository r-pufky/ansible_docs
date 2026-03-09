# Style Guide


## 80 character width
Only extend past 80 characters if long variable names make it impossible.
Continue long lines with breaks. Prefer human readability.

```yaml
- ansible.builtin.set_fact:
    JWT_SIGNING_PRIVATE_KEY_FILE:
      '{{ forgejo_config_server_app_data_path }}/jwt/private.pem',
    JWT_SIGNING_ALGORITHM: '{{
        forgejo_config_oauth2_jwt_signing_algorithm | upper
      }}',
    SSH_TRUSTED_USER_CA_KEYS_FILENAME: '{{
        _forgejo_home ~ "/.ssh/ca_trust_chain_root.pem" ~
        forgejo_config_server_ssh_trusted_user_ca_keys_filename
      }}',

# A vertical space will automatically disable jinja[spacing] test.
- ansible.builtin.set_fact:
    data_annotation: "{{ {} | r_pufky.data.v3(

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
      ) }}"
```


## Accepted Lint Disables
Isolate disable impact. Never use global lint disables for a project.

``` yaml
---
# yamllint disable rule:line-length
some really wide block text
# yamllint enable rule:line-length

some_long_yaml_line: 0  # yamllint disable-line rule:line-length
```

``` yaml
# Provide context for NOQA disables immediately.
- name: 'Annotate | sanitize & annotate service defaults'
  ansible.builtin.set_fact:  # noqa jinja[spacing] readability.
    _unifi_srv_legacy_enable: "{{ {} | r_pufky.data.v3(
        raw=unifi_srv_legacy_enable,
        ..
      ) }}"
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

### [Handler][b] (listen clauses)
Use [**listen** clause][c] to run multiple handlers for a given task.
**listen** can be a list, allowing for specific combinations as needed.

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


## File Documentation
Provide development notes, overall process, required state before executing,
post-execution state, and related information for understanding what tasks are
trying to accomplish.

A developer should be able to read this section and get an understanding of
what you are trying to do without explicitly walking through each step (that's
what tasks do!).

``` yaml
###############################################################################
# Capped Header (Context)
###############################################################################
# Immediate start description of task set.
#
# WARNING
# > Detailed end-user explanation of warning. **ALL** warnings are copied
# > verbatim to README.md.
#
# NOTE
# > Detailed explanation for developer of critical cases which hard to debug
# > failures will occur. Provide explanation, workarounds, and fixes.
#
# Requires role_other_option=true.
# ⋮Requires:
# ⋮* multi_option_one='value'.
# ⋮* multi_option_two=true.
#
# Versions (Schematic):
#       MAJOR: Unsafe - requires major role changes; only default MAJOR version
#              is supported. See other branches if they exist.
#       MINOR: Unsafe - Paperless has a history of introducing breaking changes
#              on minor updates (see 2.14, 2.15, 2.16). These should be
#              considered 'major' versions. See other branches if they exist.
#       PATCH: Safe - Usually requires no role updates. Some have included
#              breaking changes (2.16.0 - 2.16.2).
#   RECOMMEND: Only increment PATCH. Never use 'latest'.
# ⋮Always recommend and provide context for safe option changes.
#
# Security: CVE-2023-48795
#   CVE:
#   * https://nvd.nist.gov/vuln/detail/cve-2023-48795
#   * https://nvd.nist.gov/vuln/detail/cve-2023-46445
#   * https://nvd.nist.gov/vuln/detail/cve-2023-46446
#   * https://nvd.nist.gov/vuln/detail/cve-2024-41909
#   Decision: remove chacha20 cipher, *-etm MAC - disables attack for modern
#       systems, remove when OpenSSH 9.6 released with strict key exchange.
#   Mitigation:
#   * remove 'chacha20-poly1305@openssh.com' from client ciphers.
#   * remove '*-etm@openssh.com' from server macs.
#   Reference:
#   * https://terrapin-attack.com/#scanner
#   * https://www.openssh.com/txt/release-9.6
# ⋮Most critical security vulnerability first.
#
# Decision: Only manage owner - REST API does not provide a good way to set
#     user passwords (requires old and new passwords), etc; without directly
#     editing DB. Instead create required user so user can login.
# ⋮Explain opinionated or complex decision for implementing a specific way.
# ⋮Be clear and concise as to decision and supporting reasoning.
#
# Paragraph documentation.
# ⋮Make it unambiguously clear what variable does. Present, active tense,
# ⋮removing superfluous words.
# ⋮
# ⋮If it took more than a few seconds to understand write it down now.
#
# Special Case:
#         value: Description (Vertically align values, min indent 2 spaces).
#   other_value: Other.
# ⋮Special Case:
# ⋮* Non value based cases.
# ⋮* All paths are prepended with some_var.
# ⋮* Wildcards allowed.
#
# Values:
#            none: explicitly disable use of an authentication agent.
#              '': no default system agent (default).
#             ':': clarify colon usage or cases where colon could be confused.
#   SSH_AUTH_SOCK: get socket location from SSH_AUTH_SOCK environment variable.
#              $*: strings beginning with $ will be treated as environment
#                  variable containing the location of the socket.
#                      Values:
#                        {VALUE}: additional values for value element.
# ⋮All docstrings can be nested as needed for additional context.
# ⋮Always mark the default option with '(default)'.
#
# Args:
#   db: Postgres DB to manage.
# ⋮Explicitly for block/loop idioms to define what variables are required for
# ⋮task execution.
#
# Generates:
#   _paperless_ngx: dict - Role runtime specific config.
#   _paperless_ngx_map: list of dict - Annotated config map.
#   _paperless_ngx_order: list of str - Config section order.
# ⋮Runtime variables that may be consumed in other role task files.
#
# Exports:
#   _sqlite_sql_results: dict - User consumable return values from execution.
# ⋮Exported variables end-users may consume. Must update README.md.
# ⋮
# ⋮README.md:
# ⋮ ### Generated Variables
# ⋮ After successful execution the following variables are available for
# ⋮ further manipulation during the same play (standard role variable scope):
# ⋮
# ⋮  Variable            | type | Description
# ⋮ ---------------------|------|-----------------------------------------
# ⋮  _sqlite_sql_results | dict | registered return results from command.
#
# Reference:
# * https://manpages.debian.org/bookworm/openssh-client/ssh_config.5.en.html

- name: 'always use one vertical space before code'
```


## Variable Documentation
Explicitly document variable use for end users. Only remove sections which are
not relevant to variable use even if trivial. A user will **not** understand
something you implicitly do.

``` yaml
# One line active description of variable.
# ⋮Single line: Description (unit). Default: {VALUE}.
# ⋮Boolean values must declare what bool does (enable/disable).
#
# WARNING
# > Detailed end-user explanation of warning. **ALL** warnings are copied
# > verbatim to README.md.
#
# NOTE
# > Detailed explanation for developer of critical cases which hard to debug
# > failures will occur. Provide explanation, workarounds, and fixes.
#
# Requires role_other_option=true.
# ⋮Requires:
# ⋮* multi_option_one='value'.
# ⋮* multi_option_two=true.
#
# Versions (Schematic):
#       MAJOR: Unsafe - requires major role changes; only default MAJOR version
#              is supported. See other branches if they exist.
#       MINOR: Unsafe - Paperless has a history of introducing breaking changes
#              on minor updates (see 2.14, 2.15, 2.16). These should be
#              considered 'major' versions. See other branches if they exist.
#       PATCH: Safe - Usually requires no role updates. Some have included
#              breaking changes (2.16.0 - 2.16.2).
#   RECOMMEND: Only increment PATCH. Never use 'latest'.
# ⋮Always recommend and provide context for safe option changes.
#
# Security: CVE-2023-48795
#   CVE:
#   * https://nvd.nist.gov/vuln/detail/cve-2023-48795
#   * https://nvd.nist.gov/vuln/detail/cve-2023-46445
#   * https://nvd.nist.gov/vuln/detail/cve-2023-46446
#   * https://nvd.nist.gov/vuln/detail/cve-2024-41909
#   Decision: remove chacha20 cipher, *-etm MAC - disables attack for modern
#       systems, remove when OpenSSH 9.6 released with strict key exchange.
#   Mitigation:
#   * remove 'chacha20-poly1305@openssh.com' from client ciphers.
#   * remove '*-etm@openssh.com' from server macs.
#   Reference:
#   * https://terrapin-attack.com/#scanner
#   * https://www.openssh.com/txt/release-9.6
# ⋮Most critical security vulnerability first.
#
# Decision: Only manage owner - REST API does not provide a good way to set
#     user passwords (requires old and new passwords), etc; without directly
#     editing DB. Instead create required user so user can login.
# ⋮Explain opinionated or complex decision for implementing a specific way.
# ⋮Be clear and concise as to decision and supporting reasoning.
#
# Paragraph documentation.
# ⋮Make it unambiguously clear what variable does. Present, active tense,
# ⋮removing superfluous words.
# ⋮
# ⋮If it took more than a few seconds to understand write it down now.
#
# Special Case:
#         value: Description (Vertically align values, min indent 2 spaces).
#   other_value: Other.
# ⋮Special Case:
# ⋮* Non value based cases.
# ⋮* All paths are prepended with some_var.
# ⋮* Wildcards allowed.
#
# Values:
#            none: explicitly disable use of an authentication agent.
#              '': no default system agent (default).
#             ':': clarify colon usage or cases where colon could be confused.
#   SSH_AUTH_SOCK: get socket location from SSH_AUTH_SOCK environment variable.
#              $*: strings beginning with $ will be treated as environment
#                  variable containing the location of the socket.
#                      Values:
#                        {VALUE}: additional values for value element.
# ⋮All docstrings can be nested as needed for additional context.
# ⋮Always mark the default option with '(default)'.
#
# role_multi_complex:
#     list of dict - definitions for local configuration.
#   - config: str - name of local config ({INCLUDE}/{CONFIG}).
#     state: str - whether config should be managed or removed.
#           Additional description details here.
#
#           Reference:
#           * https://example.com
#         Values:
#           present: manage configuration
#            absent: remove configuration
#         Default: 'present'.
#     options: dict - ssh_config options to manage (per ssh_config options).
#       file: str - file location.
#       mode: str - octal file permissions.
#           Default: '0644'.
# ⋮For complex variables using condensed docstring format. Nest as needed.
# ⋮Always provide examples showing actual test data (see below).
#
# role_multi_complex:
#   - config: 'custom_ssh_config'
#     state: 'present'
#     options:
#       file: '/etc/example'
#       mode: '0644'
#
# role_multi_complex:
#   - config: 'alternative_config'
#     state: 'absent'
#   - config: 'custom_ssh_config'
#     state: 'present'
#     options:
#       file: '/etc/example'
#       mode: '0644'
#
# Variable: variable_name | type
# ⋮Defines an inline extended variable for a variable defined in another file.
# ⋮**Only** used for extremely large and complex configurations that have
# ⋮multiple overlapping, duplicated options, resulting in 5k+ line default
# ⋮values files.
# ⋮
# ⋮Extended options are defined in a separate file (with no
# ⋮actual YAML variables) and cross-reference actual configuration location.
# ⋮See r_pufky.deb.systemd defaults for example.
#
# Default: [] (un-managed).
# ⋮ Default: (description).
# ⋮   - 'long'
# ⋮   - 'defaults'
#
# Reference:
# * https://manpages.debian.org/bookworm/openssh-client/ssh_config.5.en.html
```


[b]: https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_handlers.html
[c]: https://github.com/ansible/ansible/issues/16378
