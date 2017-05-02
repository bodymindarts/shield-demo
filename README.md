# Shield demo

a quick introduction to shield running in habitat based docker containers.

[Shield](https://github.com/starkandwayne/shield) is a standalone system that can perform backup and restore functions for a wide variety of pluggable data systems.

You can deploy it via the [shield-boshrelease](https://github.com/starkandwayne/shield-boshrelease) or [habitat](https://www.habitat.sh/) based packages that can be found at [starkandwayne/habitat-plans](https://github.com/starkandwayne/habitat-plans)

## Install cli

the shield cli can be installed on mac via
```
brew tap starkandwayne/cf
brew install shield
```

or you can download it from the [github-release](https://github.com/starkandwayne/shield/releases)

## Demo

This demo is using the habitat based packages exported as docker containers and brought up via docker-compose.

Start the containers via
```
docker-compose up
```

Check that shield has started:
```
$ curl -k https://localhost/v1/ping
{"ok":"pong"}%
```

Create a shield backend using the shield-cli

```
$ shield create-backend local https://localhost
Successfully created backend 'local', pointing to 'https://localhost'

Using https://localhost (local) as SHIELD backend

$ shield backends
Name   Backend URI
====   ===========
local  https://localhost
```

### Creating schedule and policy

For shield to backup a _target_ some entities need to be created.

Because we are using self-signed certs we need to always add `-k` to every cli command.

Better create an alias for this demo (or it becomes very annoying)
```
$ alias shd='shield -k'
```

Lets now create all the needed entities interactively via the cli.

```
shd create-schedule

$ shd create-schedule
Schedule Name: daily
Summary:
Time Spec (i.e. 'daily 4am'): daily 4am


Schedule Name:                daily
Summary:
Time Spec (i.e. 'daily 4am'): daily 4am


Really create this schedule? [y/n] y

Created new schedule
Name:     daily
Summary:
Timespec: daily 4am

$ shd create-policy
Policy Name: short
Summary:
Retention Timeframe, in days: 1


Policy Name:                  short
Summary:
Retention Timeframe, in days: 1


Really create this retention policy? [y/n] y

Created new retention policy
Name:       short
Summary:
Expiration: 1 days
```

Now we have a schedule and a policy:

```
$ shd policies
Name   Summary  Expires in
====   =======  ==========
short           1 days
$ shd schedules
Name   Summary  Frequency / Interval (UTC)
====   =======  ==========================
daily           daily 4am
$ shd policies --raw
[{"uuid":"7d3c541b-c6f6-4c2b-9c70-ef3b7ceba7a5","name":"short","summary":"","expires":86400}]
```

The `--raw` flag gives us json output that can easily be used with the help of `jq` in scripts.

### Creating store
Next lets create a _store_ this is the place that are backups will be stored.

For the demo lets create a store using the `s3` plugin (be sure to substitute the aws credentials).

```
$ shd create-store
Store Name: default
Summary:
Plugin Name: s3
Configuration (JSON):  {"access_key_id":"<aws-access-key>","secret_access_key":"<aws-secret-key>","bucket":"shield-demo","prefix":"shield-demo"}


Store Name:           default
Summary:
Plugin Name:          s3
Configuration (JSON): {"access_key_id":"<aws-access-key>","secret_access_key":"<aws-secret-key>","bucket":"shield-demo","prefix":"shield-demo"}


Really create this archive store? [y/n] y

Created new store
Name:          default
Summary:

Plugin:        s3
Configuration: {"access_key_id":"<aws-access-key>","secret_access_key":"<aws-secret-key>","bucket":"shield-demo","prefix":"shield-demo"}
$ shd stores
Name     Summary  Plugin  Configuration
====     =======  ======  =============
default           s3      {
                            "access_key_id": "<aws-access-key>",
                            "secret_access_key": "<aws-secret-key>",
                            "bucket": "shield-demo",
                            "prefix": "shield-demo"
                          }
```

### Target

Now for the target (the thing we actually want to backup)

```
$ shd create-target
Target Name: pg-target
Summary:
Plugin Name: postgres
Configuration: {"pg_host":"target","pg_user":"admin","pg_password":"admin"}
Remote IP:port: agent:5444


Target Name:    pg-target
Summary:
Plugin Name:    postgres
Configuration:  {"pg_host":"target","pg_user":"admin","pg_password":"admin"}
Remote IP:port: agent:5444


Really create this target? [y/n] y

Created new target
Name:          pg-target
Summary:

Plugin:        postgres
Configuration: {"pg_host":"target","pg_user":"admin","pg_password":"admin"}
Remote IP:     agent:5444
$ shd targets
Name       Summary  Plugin    Remote IP   Configuration
====       =======  ======    =========   =============
pg-target           postgres  agent:5444  {
                                            "pg_host": "target",
                                            "pg_user": "admin",
                                            "pg_password": "admin"
                                          }
```
