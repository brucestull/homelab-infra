# Troubleshooting: `community.general.yaml` callback plugin removed

## Symptom

Running any playbook fails immediately with:

```
[ERROR]: The 'community.general.yaml' callback plugin has been removed. The plugin
has been superseded by the option `result_format=yaml` in callback plugin
ansible.builtin.default from ansible-core 2.13 onwards. This feature was removed
from collection 'community.general' version 12.0.0.
```

## Cause

Our `ansible.cfg` requested an old output-formatting plugin that no longer exists:

```ini
[defaults]
stdout_callback = yaml        # <-- the removed plugin
```

The `yaml` **stdout callback** used to live in the `community.general` collection.
It was removed in `community.general` 12.0.0 because the same capability is now
built into `ansible-core`'s default callback. On a new enough Ansible (this repo
uses ansible-core 2.21), the old plugin is simply gone, so Ansible errors out
instead of using it.

## Fix

Keep the **default** callback plugin and just tell it to format results as YAML,
using the modern built-in option:

```ini
[defaults]
callback_result_format = yaml   # <-- replaces stdout_callback = yaml
```

So the change is:

| Old (broken)              | New (works)                       |
| ------------------------- | --------------------------------- |
| `stdout_callback = yaml`  | `callback_result_format = yaml`   |

You still get the same readable, YAML-formatted task output — just via the
supported mechanism instead of the removed plugin.

## Why this matters (the transferable lesson)

- **Tooling evolves.** Config that worked on an older Ansible can break on a
  newer one when plugins are deprecated and removed. This is normal, not a sign
  you did something wrong.
- **Read the error message.** This one was excellent: it named the removed plugin
  *and* pointed directly at the replacement (`result_format=yaml` in the built-in
  default callback). The fix was in the error text.
- **Prefer built-ins over collection plugins for core behavior** when a built-in
  option exists — fewer moving parts, less to break across upgrades.

## Reference

- The setting lives in `ansible/ansible.cfg` under `[defaults]`.
- Discovered on: first playbook run (`playbooks/hello.yml`), ansible-core 2.21.
