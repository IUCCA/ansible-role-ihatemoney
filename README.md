# I hate money Ansible role

This is an [Ansible](https://www.ansible.com/) role which installs [I hate money](https://github.com/spiral-project/ihatemoney?tab=readme-ov-file) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

[Docher Hub Container tags](https://hub.docker.com/r/ihatemoney/ihatemoney/)

This role *implicitly* depends on:

- [`com.devture.ansible.role.playbook_help`](https://github.com/devture/com.devture.ansible.role.playbook_help)
- [`com.devture.ansible.role.systemd_docker_base`](https://github.com/devture/com.devture.ansible.role.systemd_docker_base)

For an Ansible playbook which integrates this role and makes it easier to use, see the [mash-playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).
