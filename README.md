# Docker Compose — Multi PHP Stack

A Docker Compose setup for local development with multiple PHP versions, Nginx, MySQL, Redis, HAProxy, and Supervisor — all managed through a simple CLI script.

---

## Stack

| Service | Image | Description |
|---|---|---|
| `haproxy` | HAProxy | Reverse proxy, load balancer (HTTP/HTTPS) |
| `nginx` | Nginx | Web server, routes requests to PHP-FPM |
| `php74` | PHP 7.4-fpm | FPM for `project_1`, `project_2` |
| `php81` | PHP 8.1-fpm | FPM for `project_1`, `project_2` |
| `php84` | PHP 8.4-fpm | FPM for `template-undangan`, `patradata`, `nagara` |
| `mysql` | MySQL 8.0 | Database |
| `redis` | Redis | Cache & queue driver |
| `supervisor81` | PHP 8.1 + Supervisor | Queue worker for PHP 8.1 projects |
| `supervisor84` | PHP 8.4 + Supervisor | Queue worker for PHP 8.4 projects |

### PHP Extensions (all versions)

`gd` `pdo` `pdo_mysql` `mysqli` `mbstring` `xml` `bcmath` `intl` `opcache` `pcntl` `posix` `soap` `zip` `curl` `exif` `sockets` `calendar` `gettext` `redis` `apcu`

---

## Project Structure

```
docker_compose_config/
├── dc                    # CLI helper script
├── docker-compose.yml
├── haproxy/
│   ├── Dockerfile
│   └── haproxy.cfg
├── nginx/
│   ├── Dockerfile
│   ├── nginx.conf
│   └── sites/
│       ├── project1.conf      → php81
│       ├── project2.conf      → php81
│       ├── undangan.conf      → php84
│       ├── patradata.conf     → php84
│       └── nagara.conf        → php84
├── php74/
│   ├── Dockerfile
│   └── php.ini
├── php81/
│   ├── Dockerfile
│   └── php.ini
├── php84/
│   ├── Dockerfile
│   ├── php.ini
│   └── www.conf
├── mysql/
│   ├── Dockerfile
│   ├── my.cnf
│   └── docker-entrypoint-initdb.d/
├── redis/
│   └── Dockerfile
├── supervisor81/
│   ├── Dockerfile
│   ├── supervisord.conf
│   └── conf.d/
├── supervisor84/
│   ├── Dockerfile
│   ├── supervisord.conf
│   └── conf.d/
└── logs/
    ├── nginx/
    ├── supervisor81/
    └── supervisor84/
```

---

## Requirements

- [Docker](https://docs.docker.com/get-docker/) >= 24
- [Docker Compose](https://docs.docker.com/compose/) >= 2.20 (bundled with Docker Desktop)

---

## Setup

### 1. Clone / copy this config

```bash
git clone <repo-url> docker_compose_config
cd docker_compose_config
```

### 2. Make the CLI script executable

```bash
chmod +x dc
```

### 3. Add environment variables (optional)

Create a `.env` file to override defaults:

```env
# User/Group ID (match your host user to avoid permission issues)
PUID=1000
PGID=1000

# PHP-FPM port exposed internally (no need to change)
# Nginx communicates with PHP over the Docker network

# MySQL
MYSQL_VERSION=8.0
MYSQL_DATABASE=laravel
MYSQL_USER=laravel
MYSQL_PASSWORD=secret
MYSQL_ROOT_PASSWORD=root
MYSQL_PORT=3306

# Redis
REDIS_PORT=6379

# HAProxy
HAPROXY_HTTP_PORT=8080
HAPROXY_HTTPS_PORT=8443
HAPROXY_STATS_PORT=1936

# Project paths (relative to docker_compose_config parent directory)
APP_CODE_PATH_HOST=../
```

### 4. Add local hostnames

Add entries to `/etc/hosts` (Linux/Mac) or `C:\Windows\System32\drivers\etc\hosts` (Windows):

```
127.0.0.1  project1.test
127.0.0.1  project2.test
127.0.0.1  template-undangan.test
127.0.0.1  patradata.test
127.0.0.1  nagara.test
```

### 5. Build images (first time only)

```bash
# Build a specific PHP version
./dc build php84

# Or build all at once
./dc build all
```

### 6. Start the stack

```bash
# Start with a specific PHP version
./dc up php84

# Or start everything
./dc up all
```

---

## CLI Reference (`./dc`)

```
./dc up     <php74|php81|php84|all>   Start stack
./dc kill   <php74|php81|php84|all>   Stop & remove containers
./dc restart <php74|php81|php84|all>  Restart stack
./dc status                           Show container status
./dc logs   <service>                 Tail service logs
./dc build  <php74|php81|php84|all>   Rebuild image (no cache)
./dc shell  <service>                 Open shell inside container
```

### Examples

```bash
# Start only PHP 8.4 projects (nginx + php84 + mysql + redis)
./dc up php84

# Start all PHP versions
./dc up all

# Stop only PHP 8.1 containers
./dc kill php81

# Stop everything
./dc kill all

# Rebuild PHP 8.4 image after Dockerfile changes
./dc build php84

# Watch nginx logs
./dc logs nginx

# Open a shell inside php84 container
./dc shell php84

# Open a shell inside mysql container
./dc shell mysql
```

> **Note:** `mysql` and `redis` always start regardless of the selected PHP profile.

---

## How Profiles Work

Each PHP version is isolated as a **Docker Compose profile**. Services without a profile (`nginx`, `mysql`, `redis`, `haproxy`) are always started. PHP services only start when their profile is active.

```
./dc up php84
 └─ starts: nginx, mysql, redis, php84, supervisor84
    skips:  php74, php81, supervisor81
```

This keeps resource usage low when you only need one PHP version at a time.

---

## Ports

| Port | Service |
|---|---|
| `8080` | HAProxy HTTP |
| `8443` | HAProxy HTTPS |
| `1936` | HAProxy Stats |
| `3306` | MySQL |
| `6379` | Redis |

---

## Git & `.gitignore`

A `.gitignore` is included to prevent sensitive and runtime files from being committed.

**Ignored by default:**

| Pattern | Reason |
|---|---|
| `.env`, `.env.*` | Contains passwords and credentials |
| `logs/` | Runtime log files |
| `mysql/data/` | Local database data (if using bind mount) |
| `*.pem`, `*.key`, `*.crt` | SSL/TLS certificates |
| `haproxy/certs/` | HAProxy SSL certificates |
| `.DS_Store`, `Thumbs.db` | OS-generated files |
| `.idea/`, `.vscode/` | Editor config |

**`.env.example`** — commit this as a safe template for your team:

```bash
# Create example file from your current .env
cp .env .env.example

# Then clear sensitive values in .env.example:
# MYSQL_PASSWORD=
# MYSQL_ROOT_PASSWORD=
# HAPROXY_STATS_PASSWORD=
```

When a new team member clones the repo:
```bash
cp .env.example .env
# then fill in the actual values
```

---

## Logs

Log files are written to the `logs/` directory:

```
logs/
├── nginx/            # access & error logs per site
├── supervisor81/     # queue worker logs (PHP 8.1)
└── supervisor84/     # queue worker logs (PHP 8.4)
```

---

## Troubleshooting

**Permission denied on `./dc`**
```bash
chmod +x dc
```

**Port already in use**
```bash
# Find what's using port 3306
sudo lsof -i :3306
```

**Rebuild after Dockerfile changes**
```bash
./dc build php84   # or: all
./dc restart php84
```

**Container exits immediately**
```bash
./dc logs php84
```

**MySQL data reset**
> MySQL data is persisted in a named Docker volume (`mysql-data`). To fully reset:
```bash
./dc kill all
docker volume rm docker_compose_config_mysql-data
./dc up php84
```

---

## Support

If this project has been useful to you, consider buying me a coffee!

[![Buy Me a Coffee](https://img.shields.io/badge/☕_Buy_Me_a_Coffee-Saweria-orange?style=for-the-badge)](https://saweria.co/yusril713)

Your support helps keep this maintained. Thank you!
