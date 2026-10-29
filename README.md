# HestiaCP in Docker

**Warning:** The project is still in development and may have issues. Use at your own risk.

* **[Changes](https://github.com/dddmmm5/hestiacp-docker/blob/main/CHANGES-vs-upstream.md)**
* **[Docker Hub](https://hub.docker.com/r/dddmmm/hestiacp)**
* **[Github](https://github.com/dddmmm5/hestiacp-docker)**
* **[Hestia Project](https://hestiacp.com/)**

Fork of [jhmaverick/hestiacp-docker](https://github.com/jhmaverick/hestiacp-docker) running **HestiaCP 1.10.5** on **Debian 12 (bookworm)** with Apache, Nginx, PHP-FPM (PHP 5.6–8.5), PostgreSQL, Bind, Exim, Dovecot, ClamAV and SpamAssassin.


## How to use this image

```bash
wget https://raw.githubusercontent.com/dddmmm5/hestiacp-docker/main/docker-compose.yml
wget https://raw.githubusercontent.com/dddmmm5/hestiacp-docker/main/.env.local -O .env
docker compose up -d
```

Set your hostname in `.env` (`HSTC_HOSTNAME`) before the first start.

A random password will be generated for the admin user and will be displayed in the container logs on first run (`docker logs hestiacp`). To change it:

```bash
docker exec hestiacp /usr/local/hestia/bin/v-change-user-password admin YOURNEWPASSWORD
```

**Note:** MariaDB runs in a separate container (`mariadb:10.11`) to optimize initialization time for the main container.


## Upgrading from 1.8.11

Existing containers are migrated automatically when they are recreated with the new image (`docker compose pull && docker compose up -d`). On the first start the container:

* runs the official Hestia upgrade steps from the version in the volumes to 1.10.5;
* fixes user and group IDs that changed with Debian 12;
* moves the panel to the new `hestiaweb` user;
* keeps sites on their current PHP version;
* updates the Roundcube database schema.

A backup of the configuration and the upgrade log are saved in `/root/hst_backups/`.

**Back up `/conf`, `/home`, `/backup` and the MariaDB data before upgrading.** The first start after the upgrade takes a few minutes.


## Build your own image

The `docker-helper` used in the project is just a layer for docker-compose that makes it possible to use variables for different environments, custom scripts and hooks. It depends on docker and docker-compose being installed and must be run with `bash docker-helper` or `./docker-helper`.

All configurations for building, running and pushing the images can be found in the `docker-helper.yml` in the project root.

### Build image

Run the build script informing the name of the image that will be built.
```bash
./docker-helper image-build <image>
```

Example:
```bash
./docker-helper image-build stable
```

The command exits with a non-zero code when the build fails.

### Run image

Start services using stable image:
```bash
./docker-helper up
```

Start services using another image:
```bash
./docker-helper image-up <image>
```

### Local Access
Admin URL: https://hestia.localhost:8083  
Username:  admin  
Password:  admin

The local environment (`DOCKER_ENV=local`) sets `DEV_MODE=yes`, which uses fixed passwords.

### Push image

Run the push script by entering the image name defined in `docker-helper.yml`.
```bash
./docker-helper image-push <image> <all|version|latest|<any>>
```

#### Example:
Push stable image with latest and version:
```bash
./docker-helper image-push stable all
```


## Volumes

* **/conf** Persistent data.
* **/home** All data from users.
* **/backup** Backups. Include backups from users, system credentials and daily backups from `/conf`.

### Initial data

* **/conf-start** Data for initialize `/conf`.
* **/home-start** Data for initialize `/home`.


## PHP versions

| Version | Release |
|---|---|
| PHP 5.6 | 5.6.40 |
| PHP 7.0 | 7.0.33 |
| PHP 7.1 | 7.1.33 |
| PHP 7.2 | 7.2.34 |
| PHP 7.3 | 7.3.33 |
| PHP 7.4 | 7.4.33 |
| PHP 8.0 | 8.0.30 |
| PHP 8.1 | 8.1.34 |
| PHP 8.2 | 8.2.35 |
| PHP 8.3 | 8.3.36 |
| PHP 8.4 | 8.4.27 |
| PHP 8.5 | 8.5.12 (default) |

The control panel runs on its own PHP 8.5.12.


## Security

### PHP2Shell (CVE-2026-109618, CVE-2026-109619)

Fixed in image `1.10.5.2`.

* PHP 8.2–8.5 are the fixed releases: 8.2.35, 8.3.36, 8.4.27 and 8.5.12.
* PHP 5.6–8.1 come from the [sury](https://deb.sury.org/) repository with the fixes backported (listed in each package changelog).
* The control panel PHP (`hestia-php`) is built with PHP 8.5.12. Hestia 1.10.5 ships 8.5.10, which is affected, so the image overrides it with the `HESTIA_PHP_VERSION` build argument.


## Variables

### General
* **DOCKER_REPOSITORY** Docker repository for building and pulling images.

### Container
* **HSTC_HOSTNAME** Sets the hostname of the Hestia container.
* **MAIL_ADMIN** Change mail from admin account in the first running.
* **AUTOSTART_DISABLED** Disable services on container startup. Ex: "clamav-daemon,ssh,vsftpd".

### Build
* **HESTIACP_REPOSITORY** Hestia project git that will be used in the build.
* **MULTIPHP_VERSIONS** Defines the PHP versions that will be installed in the build. Ex: `'"8.3" "8.4" "8.5"'`.
* **MARIADB_CLIENT_VERSION** Defines the version of the MariaDB client that will be installed. Default: `10.11`. Keep it on the same version as the `mariadb` image in `docker-compose.yml`: MariaDB 11.4+ clients require TLS, which a 10.11 server does not provide.
* **ZLIB_VERSION** Overrides the zlib version compiled into the Hestia packages.
* **HESTIA_PHP_VERSION** PHP version of the control panel (`hestia-php`). Default: `8.5.12`.


## Known Issues

### Build terminating for no apparent reason

Rerun the build with the `--no-cache` option. Ex: `./docker-helper image-build stable --no-cache`.

### The data directory cannot be deleted

Run `./docker-helper fix-data-chattr` and try again
