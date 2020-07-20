# dokku-gocounter

This repo contains Dockerfile and instructions for setup and running [gocounter](https://github.com/zgoat/goatcounter) as a [dokku](http://dokku.viewdocs.io/dokku/) app.

You can ignore setting up a database, but note each time you deploy a new version, all data will be lost.

## Setup

### Setup Database

#### PostgreSQL

Create a counter database:

```
psql '<connection_string>'
CREATE DATABASE counter;
CREATE USER counter WITH PASSWORD '<password>';
```

Import schema to Postgres:

```
git clone -b release-1.3 https://github.com/zgoat/goatcounter.git
psql '<connection_string>' -c '\i goatcounter/db/schema.pgsql'
```

#### SQLite

Create persistant mount:

```
dokku storage:mount <app_name> /var/lib/dokku/data/storage/<app_name>:/db
```

The connection string is now: `sqlite://db/goatcounter.sqlite3`

### Setup Dokku App

Dokku config:

```
dokku config:set <app_name> GOATCOUNTER_DB='<connection_string>'
```

Push to Dokku from local machine:

```
git remote add dokku dokku@<dokku_host>:<app_name>
git push dokku master
```

Create domain:

```
sudo docker exec <app_name>.web.1 \
    ./goatcounter create \
    -domain <domain> \
    -email <email> \
    -password <password> \
    -db '<connection_string>'
```

### Setup LetsEncrypt (optional)

```
dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git
dokku config:set --no-restart <app_name> DOKKU_LETSENCRYPT_EMAIL=<email>
dokku letsencrypt <app_name>
```

Visit <domain> in a browser you and it should all just work.

Checkout [sent-hil.com](https://sent-hil.com) to see it in action.
