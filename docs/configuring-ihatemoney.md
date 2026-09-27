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

To run an I hate money instance it is necessary to prepare a database. You can use a [MySQL](https://www.mysql.com/) compatible database server, [Postgres](https://www.postgresql.org/), or [SQLite](https://www.sqlite.org/). The SQLite database file will be automatically created by the service if it is enabled.

If you are looking for Ansible roles for a MySQL compatible server or Postgres, you can check out [ansible-role-mariadb](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb) and [ansible-role-postgres](https://github.com/mother-of-all-self-hosting/ansible-role-postgres), both of which are maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

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

Set `mysql` to use a MySQL compatible database. The SQLite database is stored in the directory specified with `ihatemoney_data_path`.

For other settings, check variables such as `ihatemoney_database_*` on [`defaults/main.yml`](../defaults/main.yml).

#### Configuring connection to the database server (optional)

By default the role is configured to establish the connection to the database server via a Unix socket. You can mount the Unix socket by adding the following configuration to your `vars.yml` file:

```yaml
# Specify the path to the MySQL compatible server's Unix socket path on the host (bind-mount source)
ihatemoney_database_mysql_socket_path_host: ""

# Specify the path to the Postgres Unix socket path on the host (bind-mount source)
ihatemoney_database_postgres_socket_path_host: ""
```

Setting it enables to connect to the database server via Unix socket mounted in the container.

If TCP connection is preferred, connection via the Unix socket can be disabled by adding the following configuration to your `vars.yml` file:

```yaml
# Disable the connection to the MySQL compatible server via a Unix socket
ihatemoney_database_mysql_socket_enabled: false

# Disable the connection to the Postgres server via a Unix socket
ihatemoney_database_postgres_socket_enabled: false
```

### Configuring the mailer (optional)

I hate money sends project invitations and password reminders by email. It is not configured to send anything until `ihatemoney_email_host` is set; setting it makes the rest of these required, because the container image writes them into a Python configuration file without quoting and an empty value stops the service from starting. The role fails the playbook run with a clear message rather than letting that happen.

```yaml
ihatemoney_email_host: mail.example.com
ihatemoney_email_port: 587
ihatemoney_host_user: ihatemoney@example.com
ihatemoney_host_password: YOUR_SMTP_PASSWORD_HERE
ihatemoney_use_tls: "1"
ihatemoney_use_ssl: "0"
ihatemoney_default_from_email: "Budget manager <ihatemoney@example.com>"
```

### Control project creation access (optional)

Out of the box, **nobody can create a project**: the role sets `ihatemoney_public_project_creation` to `false`, and `ihatemoney_admin_password` is empty. I hate money then sends anyone visiting `/create` to an administrator login that no password opens, and refuses `POST /api/projects` with a `400`. You need to pick one of the two options below before the instance is of any use.

To let anyone create a project, add the following configuration to your `vars.yml` file:

```yaml
ihatemoney_public_project_creation: true
```

To instead reserve project creation for whoever knows the administrator password, leave `ihatemoney_public_project_creation` at `false` and set `ihatemoney_admin_password` (refer to [Enabling administrative tasks](#enabling-administrative-tasks) below for how to generate its value). Setting the administrator password also enables the administration dashboard.

The two settings are independent: with `ihatemoney_public_project_creation` set to `true`, project creation stays open to everyone even while the administrator password is set.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can pass additional environment variables to the container using the `ihatemoney_container_additional_environment_variables` variable

Refer to [the official documentation](https://ihatemoney.readthedocs.io/en/latest/configuration.html) for a complete list of I hate money's config options.

Note that the container image does not read its settings from the environment directly. Its entrypoint builds `/etc/ihatemoney/ihatemoney.cfg` by interpolating a fixed list of variable names, so only a setting on that list can be reached through `ihatemoney_container_additional_environment_variables`: `DEBUG`, `ACTIVATE_ADMIN_DASHBOARD`, `ACTIVATE_DEMO_PROJECT`, `ADMIN_PASSWORD`, `ALLOW_PUBLIC_PROJECT_CREATION`, `BABEL_DEFAULT_TIMEZONE`, `MAIL_DEFAULT_SENDER`, `MAIL_PASSWORD`, `MAIL_PORT`, `MAIL_SERVER`, `MAIL_USE_SSL`, `MAIL_USE_TLS`, `MAIL_USERNAME`, `SECRET_KEY`, `SESSION_COOKIE_SECURE`, `SHOW_ADMIN_EMAIL`, `SQLALCHEMY_DATABASE_URI`, `SQLALCHEMY_TRACK_MODIFICATIONS`, `APPLICATION_ROOT`, `ENABLE_CAPTCHA` and `LEGAL_LINK`. Anything else is passed to the container and ignored.

Those values are also interpolated unquoted, so a variable that expects a Python literal (`SESSION_COOKIE_SECURE=False`, `MAIL_PORT=25`) has to be given one, and an empty value makes I hate money fail to start.

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
