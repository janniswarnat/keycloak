# Upgrade Keycloak

### Stop and remove Keycloak container

```
docker-compose rm -s -f keycloak
```

### Dump Postgres database content, e.g.

```
docker run --rm --network keycloak_default -v "$HOME/github.com/janniswarnat/keycloak/.pgpass:/root/.pgpass" postgres:15 pg_dump keycloak -h postgres -U keycloak > $HOME/github.com/janniswarnat/keycloak/keycloak-20231205.sql
```

### Stop and remove Postgres and nginx containers

```
docker-compose down
```

### Backup existing Postgres data folder, e.g.

```
sudo mv ./pgdata ./pgdata-20231205
```

### Adapt image vesions in docker-compose file and pull

```
docker-compose pull
```

### Start fresh and clean Postgres container

```
docker-compose up -d postgres
```

### Import dump into fresh container, e.g.

```
docker run --rm --network keycloak_default -v "$HOME/github.com/janniswarnat/keycloak/.pgpass:/root/.pgpass" -v "$HOME/github.com/janniswarnat/keycloak/keycloak-20231205.sql:/keycloak_dump/keycloak.sql" postgres:15 psql -h postgres -U keycloak --file /keycloak_dump/keycloak.sql
```

### Start Keycloak container and check the log if everything is ok

```
docker-compose up -d keycloak
```

### Start nginx

```
docker-compose up -d nginx
```

## More info

### Keycloak certificate

During startup of the Keycloak container a self-signed certificate is generated for localhost at `/opt/keycloak/conf/server.keystore`. Nginx by default accepts this self-signed certificate (`proxy_ssl_verify off`).

### Database update

After a version upgrade, Keycloak will update the database which then cannot easily be downgraded to lower Keycloak versions. The changes are logged in table `databasechangelog`. If you need to downgrade Keycloak, recover the database from the backup.

### Configuration

The Keycloak Docker container is now based on Quarkus. All necessary configuration is provided via Docker entrypoint. If fast startup times are needed, some of these configuration steps could be moved to the Dockerfile.
* `--auto-build`: Build in case the configuration has changed. We can use this since startup time is not relevant for our deployment.
* `--db=postgres`
* `--db-username=keycloak`
* `--db-password=<secret>`
* `--proxy=reencrypt`: Communication between reverse proxy (nginx) and Keycloak is encrypted.
* `--db-url=jdbc:postgresql://postgres/keycloak`
* `--hostname=<hostname>`
