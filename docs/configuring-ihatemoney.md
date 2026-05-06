<!--
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2020-2024 MDAD project contributors
SPDX-FileCopyrightText: 2020-2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Julian-Samuel Gebühr
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 Thomas Miceli
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara
SPDX-FileCopyrightText: 2025 MASH project contributors

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up I hate money

This is an [Ansible](https://www.ansible.com/) role which installs [I hate money](https://github.com/spiral-project/ihatemoney) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

"I hate money" is a self-hosted shared budget manager.

See the project's [documentation](https://ihatemoney.readthedocs.io/en/latest/) to learn what "I hate money" does and why it might be useful to you.

## Prerequisites

To run I hate money it is necessary to prepare a database. You can use [Postgres](https://www.postgresql.org/) or [SQLite](https://www.sqlite.org/). The SQLite database file will be automatically created by the service if it is enabled.

If you are looking for an Ansible role for Postgres, you can check out [this role (ansible-role-postgres)](https://github.com/mother-of-all-self-hosting/ansible-role-postgres) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable I hate money with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# ihatemoney                                                           #
#                                                                      #
########################################################################

ihatemoney_enabled: true

########################################################################
#                                                                      #
# /ihatemoney                                                          #
#                                                                      #
########################################################################
```

### Set the hostname

To enable I hate money you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
ihatemoney_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

### Configuring database

#### Specify database (optional)

You can specify a database used by I hate money. By default it is configured to use Postgres.

To use SQLite, add the following configuration to your `vars.yml` file:

```yaml
ihatemoney_database_type: sqlite
```

The SQLite database is stored in the directory specified with `ihatemoney_data_path`.

For other settings, check variables such as `ihatemoney_database_*` on [`defaults/main.yml`](../defaults/main.yml).

#### Configuring connection to database server

By default the role is configured to establish connection with the Postgres server via the Unix socket. You can mount the Unix socket by adding the following configuration to your `vars.yml` file:

```yaml
# Specify the path to the Postgres Unix socket path on the host (bind-mount source)
ihatemoney_database_socket_path_host: ""
```

Setting it enables to connect to the Postgres server via Unix socket mounted in the container at `/run-postgres/.s.PGSQL.5432`.

If TCP connection is preferred, connection via the Unix socket can be disabled by adding the following configuration to your `vars.yml` file:

```yaml
# Disable the connection to Postgres server via a Unix socket
ihatemoney_database_socket_enabled: false

ihatemoney_database_hostname: YOUR_POSTGRES_SERVER_HOSTNAME_HERE
ihatemoney_database_port: 5432
```

### Control project creation access (optional)

By default the instance is open to public and anyone can create a project. If you wish to limit who is capable of creating one, you can secure the instance with the admin password.

Note that the instance is automatically secured if the admin password is set to `ihatemoney_admin_password`. If you want to keep the instance public *while the admin password is set*, add the following configuration to your `vars.yml` file:

```yaml
ihatemoney_public_project_creation: true
```

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `ihatemoney_environment_variables_additional_variables` variable

See [the official documentation](https://ihatemoney.readthedocs.io/en/latest/configuration.html) for a complete list of I hate money's config options that you could put in `ihatemoney_environment_variables_additional_variables`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, I hate money becomes available at the specified hostname like `https://example.com`.

### Enabling administrative tasks

By default all administrative tasks are disabled. You can enable them by defining the `ADMIN_PASSWORD` environment variable.

You can generate an administrator's hashed password by **SSH-ing into into the server** and running a command as below:

```sh
docker exec -it ihatemoney ihatemoney generate_password_hash
```

The generated value needs to be specified to the `ihatemoney_admin_password` variable on your `vars.yml` file:

```yaml
ihatemoney_admin_password: YOUR_HASHED_PASSWORD_HERE
```

Note that the value should contain the whole output of the command, including the hashing prefix, salt and key in the format as below:

```yaml
ihatemoney_admin_password: "scrypt:32768:8:1$....$......."
```

After populating the variable, re-run the installation command.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu ihatemoney` (or how you/your playbook named the service, e.g. `mash-ihatemoney`).
