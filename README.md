# WordPress Docker Environment

A lightweight and reusable Docker environment for WordPress development and production deployments.

## Features

* WordPress with PHP-FPM 8.4
* MariaDB
* NGINX reverse proxy
* Docker Compose based
* WP-CLI helper script
* Easy environment configuration through a single `.env` file

---

## Requirements

Before getting started, make sure Docker and Docker Compose are installed.

For Ubuntu systems, an installation script is included in the root of this repository:

```bash
./install-docker.sh
```

The script follows the official Docker installation guide for Ubuntu.

---

## Configuration

Create your environment file by copying the example:

```bash
cp .env.example .env
```

Then edit the newly created `.env` file and update the following values:

| Variable                     | Description                                                                                                                                                     |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SERVICE_NAME`               | Name of your project. This is used as the Docker Compose project name.                                                                                          |
| `WORDPRESS_DB_PASSWORD`      | A strong password for the MariaDB root user.                                                                                                                    |
| `WORDPRESS_DB_NAME`          | *(Optional)* Database name. Default: `wordpress_db`.                                                                                                            |
| `WORDPRESS_DEBUG`            | *(Optional)* Set to `true` to enable WordPress debugging. Default: `false`.                                                                                     |
| `WORDPRESS_DISABLE_CRON`     | *(Optional)* Set to `false` to use built-in WP-Cron (not recommended for production). Default: `true`.                                                          |
| `WEBSITE_PROTOCOL`           | Set to `http` for local development or environments without SSL. Use `https` when SSL is handled externally (for example, by an AWS Application Load Balancer). |
| `WEBSITE_DOMAIN`             | Domain name of your website. For local development, you may use `localhost` or `127.0.0.1`.                                                                     |

---

## Database Initialization

On the first startup, the MariaDB container automatically executes any `.sql` files found in the `database/init` directory.

This is useful for restoring an existing WordPress database or providing an initial dataset for development.

Example:

```text
database/
├── data/
└── init/
    ├── 00_wordpress.sql
    └── 01_additional-data.sql
```

When the database volume is empty, all `.sql` files inside `database/init` are executed automatically in alphabetical order.

> **Note:** Initialization scripts are only executed during the first database creation. If the `database/data` directory already contains an initialized database, the scripts in `database/init` will be ignored.

---

## Starting the Environment

Start all services:

```bash
docker compose up -d
```

To stop the environment:

```bash
docker compose down
```

---

## WP-CLI

A helper script is included to simplify WP-CLI usage.

Examples:

```bash
./wp plugin list
./wp theme list
./wp core version
./wp cron event run --due-now
```

The script automatically starts a temporary WP-CLI container using the same WordPress files and database configuration.

---

## WordPress Cron

By default, the internal WordPress cron is disabled.

In production, schedule the following command (using `crontab -e`) to run every minute:

```text
* * * * * /path/to/wp cron event run --due-now
```

If you prefer to use the built-in WP-Cron (not recommended for production), set:

```text
WORDPRESS_DISABLE_CRON=false
```

---

## Project Structure

```text
.
├── database/
│   ├── data/
│   └── init/
├── nginx/
├── php/
├── www/
├── docker-compose.yaml
├── .env.example
├── install-docker.sh
└── wp
```

---

## Notes

* This environment is designed to work with both local development and production deployments.
* HTTPS certificates are **not** managed by this project. If your infrastructure provides SSL termination (such as AWS Application Load Balancer, Cloudflare, or another reverse proxy), simply set `WEBSITE_PROTOCOL=https`.
* All project-specific configuration should be managed through the `.env` file whenever possible.
