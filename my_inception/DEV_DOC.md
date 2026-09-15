# Developer Documentation

## Project layout

```
.
├── Makefile
├── README.md
├── USER_DOC.md
├── DEV_DOC.md
└── srcs/
    ├── .env
    ├── docker-compose.yml
    └── requirements/
        ├── nginx/
        │   ├── Dockerfile
        │   ├── conf/nginx.conf
        │   └── tools/entrypoint.sh
        ├── mariadb/
        │   ├── Dockerfile
        │   ├── conf/50-server.cnf
        │   └── tools/db_bootstrap.sh
        └── wordpress/
            ├── Dockerfile
            └── tools/wp_bootstrap.sh
```

## Prerequisites

* Docker Engine ≥ 20.10
* Docker Compose plugin ≥ 2.0
* Sudo access (for `make fclean` to remove host data)

## Environment setup

1. Clone the repository.
2. Edit `srcs/.env` and fill in the empty credential fields.  The file ships with dummy values.
3. Add the domain to `/etc/hosts`:

```text
127.0.0.1 hbenmoha.42.fr
```

## Makefile targets

| Target | Effect |
|--------|--------|
| `make` (or `make all`) | Create host data dirs, build images, start containers |
| `make build` | Build images without starting containers |
| `make down` | Stop and remove containers and the bridge network |
| `make clean` | Alias for `make down` |
| `make fclean` | Full teardown: containers, images, volumes, host data |
| `make re` | `fclean` followed by `all` |

## Data persistence

Named volumes are anchored to host directories via `driver_opts`:

| Volume | Container path | Host path |
|--------|---------------|-----------|
| `mariadb_persist` | `/var/lib/mysql` | `/home/hbenmoha/data/mariadb` |
| `wordpress_persist` | `/var/www/html` | `/home/hbenmoha/data/wordpress` |

A `make down` leaves these directories intact.  Only `make fclean` deletes them.

## How the entrypoints work

### MariaDB (`db_bootstrap.sh`)

1. On first start (no sentinel file), launch `mysqld_safe --skip-networking`.
2. Wait for the Unix socket to appear.
3. Create the database, application user, and set the root password.
4. Shut down the temporary instance, touch the sentinel.
5. On every start, skip straight to `exec mysqld --user=mysql`.

### WordPress (`wp_bootstrap.sh`)

1. Download WordPress core if not already present.
2. Generate `wp-config.php` with database credentials.
3. Run `wp core install` if the site is not yet installed.
4. Create a secondary author user.
5. Fix ownership to `www-data` and exec `php-fpm8.2 -F`.

### NGINX (`entrypoint.sh`)

1. Run `nginx -t` to validate the config file.
2. Exec `nginx -g "daemon off;"` so the server is PID 1.

## Networking

The `inception_net` bridge gives each container a DNS name matching its Compose service name (`mariadb`, `wordpress`, `nginx`).  No ports other than 443 are published to the host.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `mariadb` stuck in `starting` | Check `docker logs mariadb` — usually a credential mismatch or missing `/run/mysqld`. |
| WordPress shows "Error establishing a database connection" | Ensure MariaDB healthcheck passes before WordPress starts (the `depends_on` condition handles this). |
| Browser refuses to connect | Verify `hbenmoha.42.fr` resolves to `127.0.0.1` in `/etc/hosts`. |
