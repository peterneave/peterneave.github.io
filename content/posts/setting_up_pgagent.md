+++
title = 'Setting up pgAgent on Postgres'
categories = ['database','development','infrastructure']
tags = ['postgres','debian','ubuntu','linux']
date = 2025-01-29T09:00:00+11:00
+++

{{< box info >}}
  This text was originally published as a GitHub [Gist](https://gist.github.com/peterneave/83cefce2a081add244ad7dc1c53bc0c3) in December 2018.
{{< /box >}}


How to install pgAgent on Postgres on Debian-based Linux. This assumes you will have pgAgent running on the same machine as your database.

## Terminal

1. Install pgAgent via package manager

```sh
sudo apt update
sudo apt install pgagent
```

We are going to create a user to use pgAgent called `pgagent` due to [security concerns](https://www.pgadmin.org/docs/pgadmin4/dev/using_pgagent.html#security-concerns) we will run as the `postgres` user on the server and as the `pgagent` on the database.

2. Create [.pgpass file](https://www.postgresql.org/docs/current/libpq-pgpass.html).

As we are running as postgres then put it in $HOME$ for postgres (/var/lib/postgresql)

```sh
sudo su - postgres
echo localhost:5432:*:pgagent:securepassword >> ~/.pgpass
chmod 600 ~/.pgpass
chown postgres:postgres /var/lib/postgresql/.pgpass
```

3. Setup Logging directory

Setup directory for logging

```sh
mkdir /var/log/pgagent
chown -R postgres:postgres /var/log/pgagent
chmod g+w /var/log/pgagent
```

## Postgres

4. Setup pgAgent on Postgres (maintenance) Database

```sql
//use Postgres database
//This creates the pgagent schema. Dropping this extension will remove this schema and any jobs you have created.
CREATE EXTENSION pgagent;

CREATE USER "pgagent" WITH
  LOGIN
  NOSUPERUSER
  INHERIT
  NOCREATEDB
  NOCREATEROLE
  NOREPLICATION
  encrypted password 'securepassword';

GRANT USAGE ON SCHEMA pgagent TO pgagent;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA pgagent TO pgagent;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA pgagent TO pgagent;
```

### Testing

5. Test pgAgent connections

In a separate terminal, `tail -f /var/log/postgresql/postgresql-10-main.log` to watch Postgres logs

#### Postgres connection

Test connection to database in terminal with

```sh
psql -h localhost -d yourdatabase -U pgagent
```

#### pgAgent

```sh
sudo su - postgres
/usr/bin/pgagent -f -l 2 host=localhost port=5432 user=pgagent dbname=postgres
```

Review logs for errors.

### pgAgent load on boot in Debian

6. Setup pgAgent to load on reboot

Create a config file and save to /etc/pgagent.conf

```sh
#/etc/pgagent.conf
DBNAME=postgres
DBUSER=pgagent
DBHOST=localhost
DBPORT=5432
# ERROR=0, WARNING=1, DEBUG=2
LOGLEVEL=1
LOGFILE="/var/log/pgagent/pgagent.log"
```

#### Loading with Systemd

Based on the Centos pgagent.service file found at https://centos.pkgs.org/7/postgresql-12-x86_64/pgagent_12-4.0.0-4.rhel7.x86_64.rpm.html

/usr/lib/systemd/system/pgagent.service

```ini
[Unit]
Description=PgAgent for PostgreSQL
After=syslog.target
After=network.target

[Service]
Type=forking

User=postgres
Group=postgres

# Location of the configuration file
EnvironmentFile=/etc/pgagent.conf

# Where to send early-startup messages from the server (before the logging
# options of pgagent.conf take effect)
# This is normally controlled by the global default set by systemd
# StandardOutput=syslog

# Disable OOM kill on the postmaster
OOMScoreAdjust=-1000

ExecStart=/usr/bin/pgagent -s ${LOGFILE}  -l ${LOGLEVEL} host=${DBHOST} dbname=${DBNAME} user=${DBUSER} port=${DBPORT}
KillMode=mixed
KillSignal=SIGINT

Restart=on-failure

# Give a reasonable amount of time for the server to start up/shut down
TimeoutSec=300

[Install]
WantedBy=multi-user.target
```

### Start Service

```sh
sudo -i
systemctl daemon-reload
systemctl disable pgagent
systemctl enable pgagent
systemctl start pgagent
```

### Auto Rotate Logs

7. Enable auto rotation of logs

```conf
#/etc/logrotate.d/pgagent
/var/log/pgagent/*.log {
       weekly
       rotate 10
       copytruncate
       delaycompress
       compress
       notifempty
       missingok
       su root root
}
```

Test a log rotation with

```sh
logrotate -f /etc/logrotate.d/pgagent
```

### Source

- [pgAgent Install Documentation](https://www.pgadmin.org/docs/pgadmin4/dev/pgagent_install.html)
- <http://www.ekho.name/2012/03/pgagent-debianubuntu.html>
- <https://gist.github.com/ekho/6636393>
