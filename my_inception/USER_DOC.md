# User Documentation

## What is running

When the stack is up, three containers operate behind a single HTTPS endpoint:

| Container | Purpose | Internal port |
|-----------|---------|---------------|
| `nginx` | TLS termination, static file serving, PHP reverse proxy | 443 (exposed) |
| `wordpress` | PHP-FPM application server | 9000 (internal) |
| `mariadb` | MySQL-compatible database engine | 3306 (internal) |

## Starting the stack

From the repository root:

```bash
make
```

The first run downloads WordPress core, creates the database schema, and installs the site.  Subsequent starts skip these steps thanks to idempotent entrypoint scripts.

## Stopping the stack

```bash
make down
```

Containers and the bridge network are removed.  Data in `/home/hbenmoha/data` is preserved.

## Accessing the site

Open your browser and navigate to:

```
https://hbenmoha.42.fr
```

Because NGINX uses a self-signed certificate, your browser will show a security warning.  This is expected — proceed past it to reach the WordPress front page.

## WordPress admin area

```
https://hbenmoha.42.fr/wp-admin/
```

Log in with the administrator credentials defined in `srcs/.env` (`WP_ADMIN_USER` / `WP_ADMIN_PASSWORD`).  A second non-admin user (`WP_EDITOR_USER`) is also created automatically.

## Checking service health

To verify all containers are running and MariaDB reports healthy:

```bash
docker compose -f srcs/docker-compose.yml --env-file srcs/.env ps
```

You should see `mariadb` with status `healthy` and both `nginx` and `wordpress` as `Up`.

## Viewing logs

```bash
docker compose -f srcs/docker-compose.yml --env-file srcs/.env logs -f
```

Add a service name at the end to follow a single container (e.g., `logs -f nginx`).
