#  Best Practices

* Use [Data annotations](https://github.com/r-pufky/ansible_collection_data)
  for all variables touched by role consumer [See example](https://github.com/r-pufky/ansible_paperless_ngx/blob/main/tasks/annotate.yml).
* Prefix role variables with `{ROLE}_role_`.
* Prefix service variables (required to setup service, not configure it) with
  `{ROLE}_srv_`.
* Prefix config variables (required for configuring service) with
  `{ROLE}_cfg_`.
* Always create roles explicitly designed for bare-metal installations.
* Flag protect container specific options requiring explicit enablement:
    * Use a collection level flag `{COLLECTION}_container_enable`.
    * Use a role level flag `{ROLE}_container_enable`.
    * As the first step in `main.yml` override role level option if collection
      value is set ([See example](https://github.com/r-pufky/ansible_forgejo/blob/main/tasks/main.yml)).
* Handle local and remote mounted data storage possibilities:
    * Provide option for executing task as 'root' or specified user.
    * Use **UID/GID** for those locations for remote filesystem accommodation.
    * See [existing roles](https://github.com/r-pufky/ansible_paperless_ngx/blob/main/tasks/config.yml).

* [Use the style guide](style_guide.md).
* Always include last update date, version, and OS release in `vars/main.yml`:
  ``` yaml
  # Last time {ROLE} options were validated against a default configuration.
  {ROLE}_role_validate_date: '2024-06-14'
  {ROLE}_role_validate_release: 'bookworm'

  # Default packages for {ROLE}.
  {ROLE}_role_packages:
    - 'ssh'  # meta package provides both ssh and sshd.

  # Default random data.
  {ROLE}_role_generated_api_key: '{{
      lookup("ansible.builtin.password",
             "/dev/null",
             chars=["ascii_letters", "digits"],
             length=32)
    }}'
  ```

Reference:

* https://docs.ansible.com/ansible/2.8/user_guide/playbooks_best_practices.html
* https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html#magic-variables
* https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_filters.html#providing-default-values
* https://ansible.readthedocs.io/projects/lint/rules/
* https://yamllint.readthedocs.io/en/stable/rules.html
* https://docs.ansible.com/ansible/latest/dev_guide/migrating_roles.html
