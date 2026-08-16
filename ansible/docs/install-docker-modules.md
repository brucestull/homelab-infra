# `install-docker.yml` — Module Reference

A quick reference to the Ansible **module** used by each task in
`ansible/playbooks/install-docker.yml`, with a one-sentence description of what
that module does.

| # | Task | Module | What the module does |
|---|------|--------|----------------------|
| 1 | Confirm we can reach the host and see its OS | `ansible.builtin.debug` | Prints a message (or variable) to the output for information or debugging, without changing the host. |
| 2 | Install prerequisite packages | `ansible.builtin.apt` | Installs, removes, or updates Debian/Ubuntu packages to a declared state. |
| 3 | Create the apt keyrings directory | `ansible.builtin.file` | Manages a filesystem path's state and attributes (here: ensures a directory exists with given permissions). |
| 4 | Download Docker's official GPG key | `ansible.builtin.get_url` | Downloads a file from a URL to a destination path on the managed host. |
| 5 | Add the Docker apt repository (deb822 format) | `ansible.builtin.deb822_repository` | Manages an apt source in the modern deb822 `.sources` format. |
| 6 | Install Docker Engine and Compose plugin | `ansible.builtin.apt` | Installs, removes, or updates Debian/Ubuntu packages to a declared state. |
| 7 | Add the user to the docker group | `ansible.builtin.user` | Manages a user account and its group memberships. |
| H | Update apt cache (handler) | `ansible.builtin.apt` | Refreshes the apt package index; runs only when notified by a changed task. |

## Notes

- **Modules are the units of work.** Each task calls exactly one module and
  declares the *desired state*; Ansible decides whether any action is needed
  (idempotency).
- **`deb822_repository` requires `python3-debian`** on the managed host, which is
  why it's included in the prerequisites task (#2).
- **The handler** (`Update apt cache`) is triggered via `notify` from the
  repository task and runs once at the end of the play — only if that task
  actually changed something.
