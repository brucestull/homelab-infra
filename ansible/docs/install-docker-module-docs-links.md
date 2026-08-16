# `install-docker.yml` — Official Module Documentation Links

Official Ansible documentation for each **module** used in
`ansible/playbooks/install-docker.yml`. All are part of `ansible.builtin`
(shipped with every Ansible install).

| Module | What it does | Official docs |
|--------|--------------|---------------|
| `ansible.builtin.debug` | Prints statements/variables during execution. | https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/debug_module.html |
| `ansible.builtin.apt` | Manages Debian/Ubuntu (apt) packages. | https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/apt_module.html |
| `ansible.builtin.file` | Manages files, directories, symlinks, and their properties. | https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/file_module.html |
| `ansible.builtin.get_url` | Downloads files from HTTP, HTTPS, or FTP to the node. | https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/get_url_module.html |
| `ansible.builtin.deb822_repository` | Adds/removes apt repositories in the deb822 `.sources` format. | https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/deb822_repository_module.html |
| `ansible.builtin.user` | Manages user accounts and group memberships. | https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/user_module.html |

## Handy top-level references

| Resource | Link |
|----------|------|
| Index of all `ansible.builtin` modules | https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/index.html |
| Ansible documentation home | https://docs.ansible.com/ |

## Notes

- These links point at the `latest` docs. To match the exact version this repo
  runs, replace `latest` with the ansible-core version (e.g. a specific `2.x`)
  in the URL path if you need version-pinned documentation.
- The older `docs.ansible.com/ansible/latest/...` URL form still works and
  redirects to the `projects/ansible/latest/...` paths used above.
- Pairs with `install-docker-modules.md` (one-line descriptions per task).
