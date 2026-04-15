# Best Practices
!!! tip "Prefer consistency in documentation and update if file is touched."
    Always resolve yamllint, and ansible-lint issues with **no** global
    exceptions ([example][a]).

!!! success "Use ansible sourced jinja2 templates for release cycles faster than Debian"
    Many modern applications use rolling release or rapid release cycles in the
    span of months ➔ weeks ➔ days. Many of these updates have breaking config
    setting changes requiring complete re-evaluation of application under
    management when encapsulating configuration settings.

    This is untenable for maintaining large numbers of roles.

    **Any** application where major releases (or breaking releases) occur
    [faster than the Debian release cycle][d] **must use** ansible source
    jinja2 templates for configuration. This enables the role to focus on
    critical setup and the consumer to configure the application settings
    appropriately.

    See [Templated Directories](patterns.md#templated-directories).

* Use [Data annotations][b] for variables touched by consumer ([example][a])
  that require data manipulation for role application.
* Always create roles explicitly designed for bare-metal installations.
* Use feature flags to stage role execution steps.
* Provide debug notification & estimated wait time with [glyphs](#text-glyphs)
  for long running tasks ([example][c]).
* Always include update date, version, and OS release in **vars/main.yml**:
  ``` yaml
  {ROLE}___release: "{{ {} | r_pufky.data.v3(

      _date='2024-06-14',
      _debian='bookworm',
      _app='1.20.20',
    )}}"

  {ROLE}___packages:
    - 'ssh'
  ```

## Format Rules
See [style guide](style_guide.md).

  Style  | Rule
 -------:|----------------------------------------------------------------
  **80** | Character width.
  **2**  | Spaces per tab.
  **2**  | Spaces - nested section.
  **4**  | Spaces - extended text from previous line.
  **1**  | Vertical space between sections.
  **.**  | All lines end in `.` for complete statement unless exactly 80.
  **#**  | Comment - all lines end in `.` unless exactly 80.
  Tense  | Always use present active tense for concise documentation.


## Naming Conventions
 Style                  | Rule
-----------------------:|-----------------------------------------------------------
**`var_name`**          | Variables are unquoted in paragraphs.
**`{ROLE}___{VAR}`**    | Role variable.
**`{ROLE}___{VAR}`**    | Role runtime variable.
**`^$*{VAL}`**          | regex expressions are allowed; use {VAL}
**`N/A`**               | not applicable
**`':'`**               | Always quote to disambiguate separators.


## Abbreviated Data Types
 Type                   | Default
-----------------------:|--------------------------------------------------
 **`int`**              | **`#`** (not empty).
 **`float`**            | **`#.#`** (not empty).
 **`bool`**             | **`True`** or **`False`** (not empty).
 **`str`**              | **`''`**
 **`list`**             | **`[]`**
 **`dict`**             | **`{}`**
 **`{TYPE} of {TYPE}`** | Outer most type first (e.g. list of str - **`[]`**).
 **`{TYPE} or {TYPE}`** | Multiple types - preferred type first.

## TODO
Place anywhere where additional work is needed. No restrictions.

``` yaml
# TODO(bug): List deletion via API is currently broken. This is fixed in the
#     next PiHole release. Lists will still be managed, but cannot be deleted.
# ⋮ Good TODO categories: bug, role, user, {VERSION}.
```

## ⋮ Alternative Context
Include alternative docstring format or context inline with the docstring
option. This provides self-documenting expanded use features.

``` yaml
# Requires role_other_option=true.
# ⋮Requires:
# ⋮* multi_option_one='value'.
# ⋮* multi_option_two=true.
```

## Text Glyphs

 Glyph | Code  | Use
------:|:------|--------------------------
 ➔     | 2794  | Menus, sub-items, links.
 🛆     | 26a0  | Warning.
 ⓘ     | 24be  | Informational.
 🗘    | 1f5d8 | Waiting.
 ✔     | 2714  | Success / Enabled.
 ✘     | 2718  | Failure / Disabled.
 ⋮     | 22ee  | Additional context (or context menu).
 ⚙     | 2699  | Settings.
 ⌘     | 2318  | Super key.

## Reference[^1][^2][^3][^4][^5][^6]
[^1]: https://docs.ansible.com/ansible/2.8/user_guide/playbooks_best_practices.html
[^2]: https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html#magic-variables
[^3]: https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_filters.html#providing-default-values
[^4]: https://ansible.readthedocs.io/projects/lint/rules/
[^5]: https://yamllint.readthedocs.io/en/stable/rules.html
[^6]: https://docs.ansible.com/ansible/latest/dev_guide/migrating_roles.html

[a]: https://github.com/r-pufky/ansible_paperless_ngx
[b]: https://github.com/r-pufky/ansible_collection_data
[c]: https://github.com/r-pufky/ansible_paperless_ngx/blob/main/tasks/install.yml#L292
[d]: https://www.debian.org/releases
