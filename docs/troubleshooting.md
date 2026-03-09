# General Troubleshooting

## [Remove Submodule][a]
Usually after a bad initial creation or weird sync state.

``` bash
# Remove from collection root
rm -rf roles/{ROLE}  # git rm -r roles/{ROLE}
rm -rf .git/modules/roles/{ROLE}
git config --remove-section submodule.roles/{ROLE}

# Submodule is fully removed and ready to be re-added.
```


## Multiple [configurations found for submodule][b]
**.git/config** mis-match against checked in submodules file **.gitmodules**.

!!! danger ""

    ``` bash
    warning: {HASH}:.gitmodules, multiple configurations found for 'submodule.roles/{ROLE}'.
    Skipping second one!
    ```

Ensure files are the same. Check both **.git/config** and **.gitmodules** are
up to date and the same. Add to git commit if needed.


## no_log does not [honor variable use][c]
`no_log` currently does not honor variable interpretation.

Use `loop_control` whenever possible or statically set `no_log: true`
```yaml
- name: 'looping task with passwords'
  ansible.builtin.include_tasks: 'some_task.yml'
  loop: '{{ user_accounts }}'
  loop_control:  # Preferred - provides execution context.
    label: '{{ item.user }}'

- name: 'looping task with passwords'
  ansible.builtin.include_tasks: 'some_task.yml'
  no_log: true  # Strips all debugging feedback.
```

[a]: https://stackoverflow.com/questions/1260748/how-do-i-remove-a-submodule
[b]: https://stackoverflow.com/questions/51883100/git-submodule-warning-multiple-configurations-found
[c]: https://github.com/ansible/ansible/issues/83323#issuecomment-2686201726