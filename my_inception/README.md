*This project has been created as part of the 42 curriculum by hbenmoha.*

# Inception

## Description

Inception is a Docker-based mini-infrastructure that runs a WordPress site behind an NGINX TLS termination proxy with a MariaDB backend.  Every container is built from scratch on top of `debian:bookworm`, no ready-made service images are used.

The stack consists of three services, each with its own Dockerfile under `srcs/requirements/`:

| Service | Role |
|---------|------|
| **NGINX** | The sole entry point.  Listens on port 443 with TLS 1.2/1.3 and reverse-proxies PHP requests to WordPress. |
| **WordPress** | Runs PHP-FPM and serves the WordPress application.  The entrypoint downloads core, creates `wp-config.php`, installs the site, and creates two users. |
| **MariaDB** | Stores the WordPress database.  A one-shot bootstrap creates the schema and application user on first start. |

A private Docker bridge network (`inception_net`) connects the three containers.  Two named volumes , `mariadb_persist` and `wordpress_persist` , are mapped to `/home/hbenmoha/data` on the host for persistence.

## How to use

### Prerequisites

* A Linux host with Docker Engine and the Compose plugin.
* Add the domain to `/etc/hosts`:

```text
127.0.0.1 hbenmoha.42.fr
```

### Start the stack

```bash
make
```

This creates the host data directories, builds the images, and starts all three containers.

### Stop (data preserved)

```bash
make down
```

### Full teardown (data destroyed)

```bash
make fclean
```

## Key comparisons

### Virtual machines vs Docker containers

A virtual machine bundles a full guest operating system and runs an entire kernel on top of a hypervisor.  Docker containers share the host kernel and isolate only the application layer using namespaces and cgroups.  The result is containers start in seconds rather than minutes, consume far less RAM, and can be ephemeral by design.

### Environment variables vs secrets

Environment variables are simple key-value pairs injected into a container's runtime.  They work well for non-sensitive configuration (port numbers, domain names).  Docker secrets, by contrast, are mounted as temporary files under `/run/secrets/` and are never visible in `docker inspect` output or process listings.  For credentials , database passwords, API keys , secrets are the safer choice.  This project uses an `.env` file for simplicity, but the same variables could be migrated to Docker secrets with minimal changes.

### Named volumes vs bind mounts

A bind mount exposes an absolute host path directly into the container (e.g., `-v /home/user/data:/data`).  It is easy to debug but couples the container to the host's filesystem layout.  Named volumes are managed by Docker's volume driver and stored under Docker's data root by default.  They can be backed by a bind mount for explicit host placement (as this project does) while still benefiting from Docker's lifecycle management.  Volumes survive `docker compose down`; bind mounts survive too, but they are not tracked as Docker objects.

## Resources

* [Docker documentation](https://docs.docker.com/)
* [Docker Compose specification](https://docs.docker.com/compose/compose-file/)
* [NGINX TLS configuration](https://nginx.org/en/docs/http/configuring_https_servers.html)
* [MariaDB knowledge base](https://mariadb.com/kb/)
* [WordPress documentation](https://developer.wordpress.org/)
