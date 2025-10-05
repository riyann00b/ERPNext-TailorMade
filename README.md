# 🚀 ERPNext-TailorMade: A Guide to Custom Docker Installation

A comprehensive guide to installing a full ERPNext stack with Docker, including custom apps and an NGINX reverse proxy for local HTTPS. This document details a production-inspired setup, including post-installation workarounds and instructions for enabling Google Integration with a self-signed certificate.

> [!Note]
> ### 🙏 Acknowledgements
> This guide is heavily inspired by Atul Bhatt's YouTube video: [Frappe Docker Custom Apps Installation using Quick Build Image | 2025](https://www.youtube.com/watch?v=FWGWKC_rZeI).
> The official Frappe Docker documentation is the primary source. Always check for updates there before following this guide: [Official Frappe Docker Documentation](https://github.com/frappe/frappe_docker/tree/main).

---

## ✅ Prerequisites

Ensure the following software is installed and running:
- **Docker**
- **Docker Compose**
- **Git**
- **OpenSSL** (for generating self-signed SSL certificates)
- **Optional**: `jq` (for JSON validation), `nano` or `VS Code` (for editing files)
- **Nginx**

## Table of Contents

- [🚀 ERPNext-TailorMade: A Guide to Custom Docker Installation](#-erpnext-tailormade-a-guide-to-custom-docker-installation)
- [Prerequisites](#-prerequisites)
- [Simple Setup (No Custom Apps)](#self-hosted-docker-container-simple-setup-no-custom-apps)
- [Complex Setup with Custom Apps](#self-hosted-docker-container-more-complex-setup-with-custom-apps)
- [Complex Setup with Custom Apps, HTTPS, Custom Domain, and SSL](#self-hosted-docker-container-complex-setup-with-custom-apps-https-custom-domain-and-ssl
)
- [Backup Settings, Inventory, and Updating ERPNext](#backup-settings-inventory-and-updating-erpnext)
  - [Backup Steps](#backup-settings-inventory-and-updating-erpnext)
  - [Restore Steps](#restore-steps)
- [Automatic Sync to Google Drive](#automatic-sync-to-google-drive)
- [Troubleshoot](#troubleshoot)
  - [OSError: wkhtmltopdf reported an error](#oserror-wkhtmltopdf-reported-an-error-exit-with-code-1-due-to-network-error-connectionrefused)

[Post install guide]()

[my n8n setup](https://github.com/riyann00b/ERPNext-TailorMade/blob/main/n8n%20setup.md)

---

### Self-Hosted Docker Container: Simple Setup (No Custom Apps)

See [Self-hosted Docs](https://github.com/frappe/erpnext#self-hosted).

#### Docker Installation

See [ERPNext Docker Docs](https://github.com/frappe/erpnext#docker).

> [!NOTE]
> **Prerequisites**: Docker, Docker Compose, Git. Refer to [Docker Documentation](https://docs.docker.com) for more details on Docker setup.

Run the following commands:

```bash
git clone https://github.com/frappe/frappe_docker
cd frappe_docker
docker compose -f pwd.yml up -d
```

After a couple of minutes, the site should be accessible on your localhost port `8080`. Use the default login credentials to access the site:
- **Username**: `Administrator`
- **Password**: `admin`

See [Frappe Docker](https://github.com/frappe/frappe_docker?tab=readme-ov-file#to-run-on-arm64-architecture-follow-this-instructions) for ARM64 architecture Docker setup.

#### To run on ARM64 architecture follow these instructions:

See [ARM64 Docs](https://github.com/frappe/frappe_docker?tab=readme-ov-file#to-run-on-arm64-architecture-follow-this-instructions).

After cloning the repo and `cd frappe_docker`, run this command to build multi-architecture images specifically for ARM64:

```bash
docker buildx bake --no-cache --set "*.platform=linux/arm64"
```

Then:
- Add `platform: linux/arm64` to all services in the `pwd.yml`.
- Replace the current specified versions of the `erpnext` image in `pwd.yml` with `:latest`.

Finally, run:

```bash
docker compose -f pwd.yml up -d
```

---

## Self-Hosted Docker Container: More Complex Setup with Custom Apps

### Building the Custom Docker Image

#### Step 1: Clone the Frappe Docker Repository
Clone the official `frappe_docker` repository and navigate into the `development` directory.

```bash
git clone https://github.com/frappe/frappe_docker
cd frappe_docker/development
```

#### Step 2: Configure Custom Applications via `apps.json`
Create a file named `apps.json` that lists all the apps to be installed.

First, remove the example file:

```bash
rm -f apps-example.json
```

Then, create and edit the new file using your preferred editor:

```bash
# Using nano
nano apps.json

# Or using VS Code
code apps.json
```

**`apps.json` Example:**

```json
[
  {
    "url": "https://github.com/frappe/erpnext",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/ecommerce_integrations.git",
    "branch": "main"
  },
  {
    "url": "https://github.com/resilient-tech/india-compliance",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/print_designer.git",
    "branch": "develop"
  },
  {
    "url": "https://github.com/clefincode/clefincode_chat.git",
    "branch": "develop"
  }
]
```

> [!IMPORTANT]
> **Dependencies**: You must manually add app dependencies. For example, `hrms` requires `erpnext` to be in the list.
> **Private Repos**: For private repositories, use an HTTPS git URL with a Personal Access Token (PAT): `https://{{PAT}}@github.com/your-org/your-private-app.git`.

#### Step 3: Encode `apps.json` to Base64
The Docker build process requires the `apps.json` content to be passed as a Base64 encoded string.

> [!NOTE]
> Ensure you are in the `frappe_docker/development` directory.

```bash
export APPS_JSON_BASE64=$(base64 -w 0 apps.json)
```

To validate the string, you can decode it and check the contents:

```bash
# Decode the variable into a test file and display it
echo -n ${APPS_JSON_BASE64} | base64 -d > apps-test-output.json
cat apps-test-output.json

# Optional: Validate the original JSON syntax (requires jq)
jq empty apps.json
```

#### Step 4: Build and Push the Custom Docker Image
This is a multi-stage process to build, tag, and push your image.

##### 🏗️ A. Build the Image

> [!NOTE]
> Navigate back to the root `frappe_docker` directory.

```bash
cd ..
```

Run the build command using the **Quick Build** method for faster builds.

**Quick Build Command Example:**

Refer to [configure-build](https://github.com/frappe/frappe_docker/blob/main/docs/custom-apps.md#configure-build) for full details.

> [!TIP]
> Change the `tag` to your Docker Hub `username` and tag the image as `latest`.

```bash
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=riyann00b/frappe-custom:latest \
  --file=images/layered/Containerfile .
```

> [!IMPORTANT]
> ☕ Sit back, relax, and have a coffee. This will take a while.

Verify the new image:

```bash
docker images
```

#### Step 5: Docker Compose Configuration and Launch

##### Create the Final `pwd.yml` Docker Compose File

> [!TIP]
> Navigate to the `frappe_docker` directory if not already there:

```bash
cd /path/to/your/frappe_docker
# or
cd ..
```

Edit the `pwd.yml` file with your preferred text editor:

```bash
# Using nano
sudo nano pwd.yml

# Or using VS Code
code pwd.yml
```

> [!TIP]
> -   Replace all instances of the default image (e.g., `frappe/erpnext:v15.xx.x`) with your custom image (e.g., `riyann00b/frappe-custom:latest`).
> -   Add `platform: linux/amd64` to all services in the `pwd.yml` file to ensure compatibility (e.g., for Linux, Apple M1/M2 systems).
> -   Update the `create-site` service's command to include your custom apps for installation.

**Example modification in `pwd.yml`:**

```yaml
  create-site:
    command:
        bench new-site --mariadb-user-host-login-scope='%' --admin-password=admin --db-root-username=root --db-root-password=admin --install-app erpnext '--install-app YOUR-CUSTOM-APPs-name' --set-default frontend;
```

See example `pwd.yml` file [here](https://github.com/riyann00b/ERPNext-TailorMade/blob/main/pwd.yml).

> [!TIP]
> Apps to copy for the `--install-app` flag:

```bash
--install-app ecommerce_integrations --install-app india_compliance --install-app print_designer --install-app clefincode_chat
```

#### Step 6: Launch the Stack and Finalize Setup

1.  **Launch all services:**
    From the root `frappe_docker` directory, run:

    ```bash
    docker compose -f pwd.yml up -d
    ```

2.  **Monitor site creation**:

    ```bash
    docker logs frappe_docker-create-site-1 -f
    ```

3.  **Access your site:**
    Open your browser and navigate to [http://localhost:8080](http://localhost:8080).

---

## Self-Hosted Docker Container: Complex Setup with Custom Apps, HTTPS, Custom Domain, and SSL

### Part 1: Building the Custom Docker Image

This part is **identical** to the previous guide. The goal is to create your custom image with your specific apps. If you have already built your `riyann00b/frappe-custom:latest` image, you can **skip to Part 2**.

#### Step 1: Clone the Frappe Docker Repository
```bash
git clone https://github.com/frappe/frappe_docker
cd frappe_docker
```

#### Step 2: Configure Custom Applications via `apps.json`
Create a file named `apps.json` inside the `development` directory.

```bash
cd development
nano apps.json
```

**`apps.json` Example:**

```json
[
  {
    "url": "https://github.com/frappe/erpnext",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/hrms",
    "branch": "version-15"
  }
]
```

#### Step 3: Encode `apps.json` to Base64
```bash
export APPS_JSON_BASE64=$(base64 -w 0 apps.json)
```

#### Step 4: Build the Custom Docker Image
Navigate back to the root `frappe_docker` directory to build the image.

```bash
cd ..
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=riyann00b/frappe-custom:latest \
  --file=images/layered/Containerfile .
```

---

### Part 2: Automated Deployment with Traefik

This is where the magic happens. We will define our entire stack, including the reverse proxy and SSL, in configuration files.

#### Step 5: The All-Important `.env` Configuration
Create an `.env` file in your `frappe_docker` directory. This file provides the two critical pieces of information Traefik needs to automate HTTPS.

```bash
# In the frappe_docker directory
nano .env
```

Paste the following, replacing the values with your own domain and email:

```dotenv
# The domain name(s) for your ERPNext site, separated by commas
SITES=`erp.yourdomain.com`

# The email address for your Let's Encrypt SSL certificate
LETSENCRYPT_EMAIL=your.email@example.com

# Your custom image details
CUSTOM_IMAGE=riyann00b/frappe-custom
CUSTOM_TAG=latest

# Your database password
DB_PASSWORD=a-very-secure-password
```

#### Step 6: Generate the Docker Compose File with Traefik
Now, we'll generate a single `docker-compose.yml` file that includes the Traefik proxy service.

```bash
# Create a directory to store your final configurations
mkdir ~/gitops

# Generate the final docker-compose.yml file
docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.https.yaml \
  config > ~/gitops/docker-compose.yml
```

> [!Note]
> ### How It Works: The Magic of Docker Labels
> The `overrides/compose.https.yaml` file adds a `proxy` service (Traefik) and adds `labels` to the `frontend` service. These labels are instructions that Traefik reads automatically:
> -   `traefik.enable=true`: Tells Traefik to manage this container.
> -   `traefik.http.routers.frontend-http.rule=Host(${SITES})`: Tells Traefik to route traffic for the domain in your `.env` file to this container.
> -   `traefik.http.routers.frontend-http.tls.certresolver=main-resolver`: Tells Traefik to automatically get an SSL certificate for this domain.

---

### Part 3: Launch and Finalize

#### Step 7: Launch the Entire Stack
From the `~/gitops` directory, start everything with one command. Docker Compose will read your `.env` file, pull the necessary images, and start all containers, including Traefik.

```bash
cd ~/gitops
docker compose up -d
```

Traefik will now start, detect the `frontend` container, and automatically request an SSL certificate from Let's Encrypt in the background.

#### Step 8: Create the First Site
This final step is crucial. You must create a site whose name **exactly matches** the domain you specified in your `.env` file.

```bash
# From the ~/gitops directory
docker compose exec backend bench new-site \
  --mariadb-user-host-login-scope=% \
  --db-root-password 'a-very-secure-password' \
  --admin-password 'your-admin-password' \
  --install-app erpnext \
  --install-app hrms \
  erp.yourdomain.com
```

#### Step 9: Access Your Secure, Custom Site
That's it! Open your browser and navigate to your domain. You have a fully custom ERPNext installation running securely over HTTPS with zero manual proxy or SSL configuration.

-   **URL**: `https://erp.yourdomain.com`

---

## Backup Settings, Inventory, and Updating ERPNext

### Backup Steps

1.  **Set variables/placeholders**:

    ```bash
    COMPOSE_YML=/path/to/pwd.yml
    SITE=your_site_name
    DEST=/on/host/backup/destination
    DB_NAME=your_database_name
    ```

2.  **Dump the MariaDB database**:

    ```bash
    docker compose -f $COMPOSE_YML exec db sh -c \
      'exec mariadb-dump --single-transaction -uroot -p"$MYSQL_ROOT_PASSWORD" '$DB_NAME \
      ' ' | gzip > $DEST/database-$(date +%Y%m%d_%H%M%S).sql.gz
    ```

3.  **Back up the `site_config.json`**:

    ```bash
    docker compose -f $COMPOSE_YML cp \
      backend:/home/frappe/frappe-bench/sites/$SITE/site_config.json \
      $DEST/site_config-$(date +%Y%m%d_%H%M%S).json
    ```

4.  **Back up private file uploads**:

    ```bash
    docker compose -f $COMPOSE_YML exec backend sh -c \
      'exec tar cf - ./sites/'$SITE'/private/files' | gzip > \
      $DEST/Private-Files-$(date +%Y%m%d_%H%M%S).tar.gz
    ```

5.  **Back up public file uploads**:

    ```bash
    docker compose -f $COMPOSE_YML exec backend sh -c \
      'exec tar cf - ./sites/'$SITE'/public/files' | gzip > \
      $DEST/Public-Files-$(date +%Y%m%d_%H%M%S).tar.gz
    ```

---

### Restore Steps

1.  **Set variables/placeholders**:

    ```bash
    COMPOSE_YML=/path/to/pwd.yml
    SITE=your_site_name
    DB_NAME=your_database_name
    SOURCE=/on/host/backup/source
    ```

2.  **Decompress the database dump**:

    ```bash
    gzip -d $SOURCE/database-date_time_of_backup.sql.gz
    ```

3.  **Copy the SQL file into the `db` container**:

    ```bash
    docker compose -f $COMPOSE_YML cp \
      $SOURCE/database-date_time_of_backup.sql \
      db:/tmp/
    ```

4.  **Restore database inside container**:

    ```bash
    docker compose exec -i db sh -c \
      'exec mariadb -uroot -p"$MYSQL_ROOT_PASSWORD" '$DB_NAME' < /tmp/database-date_time_of_backup.sql'
    ```

5.  **Restore `site_config.json`** (if needed, e.g., for encryption keys) by copying the saved version over the live one.

6.  **Copy file archives into `backend`**:

    ```bash
    docker compose -f $COMPOSE_YML cp \
      $SOURCE/Public-Files-date_time_of_backup.tar.gz backend:/home/frappe/frappe-bench
    docker compose -f $COMPOSE_YML cp \
      $SOURCE/Private-Files-date_time_of_backup.tar.gz backend:/home/frappe/frappe-bench
    ```

7.  **Unpack the archives inside the `backend` container**:

    ```bash
    docker compose -f $COMPOSE_YML exec backend tar xzf Public-Files-date_time_of_backup.tar.gz
    docker compose -f $COMPOSE_YML exec backend tar xzf Private-Files-date_time_of_backup.tar.gz
    ```

---

If you like, I can give you a **ready bash script** using your `pwd.yml` and actual app names to automate this.

## Automatic Sync to Google Drive

Your current backup and restore steps are correct. To extend them with **automatic sync to Google Drive**, you only need to wrap them in a script and add `rclone`.

**1. Install rclone and configure Google Drive**

```bash
sudo apt install rclone
rclone config   # create remote called "gdrive"
```

**2. Create backup script `/srv/scripts/erpnext_backup.sh`**

```bash
#!/bin/bash
set -e

COMPOSE_YML=/path/to/pwd.yml
SITE=your_site_name
DB_NAME=your_database_name
DEST=/srv/backups

mkdir -p $DEST

# Database
docker compose -f $COMPOSE_YML exec db sh -c \
  'exec mariadb-dump --single-transaction -uroot -p"$MYSQL_ROOT_PASSWORD" '$DB_NAME \
  ' ' | gzip > $DEST/database-$(date +%Y%m%d_%H%M%S).sql.gz

# site_config.json
docker compose -f $COMPOSE_YML cp \
  backend:/home/frappe/frappe-bench/sites/$SITE/site_config.json \
  $DEST/site_config-$(date +%Y%m%d_%H%M%S).json

# Private files
docker compose -f $COMPOSE_YML exec backend sh -c \
  'exec tar cf - ./sites/'$SITE'/private/files' | gzip > \
  $DEST/Private-Files-$(date +%Y%m%d_%H%M%S).tar.gz

# Public files
docker compose -f $COMPOSE_YML exec backend sh -c \
  'exec tar cf - ./sites/'$SITE'/public/files' | gzip > \
  $DEST/Public-Files-$(date +%Y%m%d_%H%M%S).tar.gz

# Delete older than 14 days
find $DEST -type f -mtime +14 -delete

# Sync to Google Drive
rclone copy $DEST gdrive:/erpnext-backups --transfers=1 --checkers=2
```

**3. Make executable**

```bash
chmod +x /srv/scripts/erpnext_backup.sh
```

**4. Add cron job**

```bash
crontab -e
0 2 * * * /srv/scripts/erpnext_backup.sh >> /var/log/erpnext_backup.log 2>&1
```

This keeps local backups for 14 days and pushes them to Google Drive every night.

Do you want me to also add a **restore-from-Google-Drive script** so you can pull any backup directly without manual download?

### Restore from Google Drive Script

Here’s a restore script that pulls your backups directly from Google Drive using `rclone`:

**1. Create `/srv/scripts/erpnext_restore.sh`**

```bash
#!/bin/bash
set -e

# Variables
COMPOSE_YML=/path/to/pwd.yml
SITE=your_site_name
DB_NAME=your_database_name
RESTORE_DIR=/srv/restore
REMOTE=gdrive:/erpnext-backups   # rclone remote

mkdir -p $RESTORE_DIR

echo "Fetching latest backup from Google Drive..."
rclone copy --max-age 30d --include "database-*.sql.gz" $REMOTE $RESTORE_DIR --progress
rclone copy --max-age 30d --include "site_config-*.json" $REMOTE $RESTORE_DIR --progress
rclone copy --max-age 30d --include "Private-Files-*.tar.gz" $REMOTE $RESTORE_DIR --progress
rclone copy --max-age 30d --include "Public-Files-*.tar.gz" $REMOTE $RESTORE_DIR --progress

# Identify latest files
DB_FILE=$(ls -t $RESTORE_DIR/database-*.sql.gz | head -1)
SITE_CONFIG=$(ls -t $RESTORE_DIR/site_config-*.json | head -1)
PRIVATE_FILES=$(ls -t $RESTORE_DIR/Private-Files-*.tar.gz | head -1)
PUBLIC_FILES=$(ls -t $RESTORE_DIR/Public-Files-*.tar.gz | head -1)

echo "Restoring database..."
gzip -d -c $DB_FILE > $RESTORE_DIR/database.sql
docker compose -f $COMPOSE_YML cp $RESTORE_DIR/database.sql db:/tmp/database.sql
docker compose -f $COMPOSE_YML exec -T db sh -c \
  'exec mariadb -uroot -p"$MYSQL_ROOT_PASSWORD" '$DB_NAME' < /tmp/database.sql'

echo "Restoring site_config.json..."
docker compose -f $COMPOSE_YML cp $SITE_CONFIG backend:/home/frappe/frappe-bench/sites/$SITE/site_config.json

echo "Restoring public/private files..."
docker compose -f $COMPOSE_YML cp $PRIVATE_FILES backend:/home/frappe/frappe-bench/
docker compose -f $COMPOSE_YML cp $PUBLIC_FILES backend:/home/frappe/frappe-bench/
docker compose -f $COMPOSE_YML exec backend tar xzf $(basename $PRIVATE_FILES)
docker compose -f $COMPOSE_YML exec backend tar xzf $(basename $PUBLIC_FILES)

echo "Restore completed."
```

**2. Make it executable**

```bash
chmod +x /srv/scripts/erpnext_restore.sh
```

**3. Usage**
When needed:

```bash
/srv/scripts/erpnext_restore.sh
```

This will always pull the most recent database, config, and file archives from Google Drive, then restore them inside your containers.

Do you also want me to add **option flags** (e.g. `--date 20250910` to restore a specific day) instead of always restoring the latest?

---

## Troubleshoot

### OSError: wkhtmltopdf reported an error: exit with code 1 due to network error: connectionrefused

```bash
docker exec -u 0 -it  frappe_docker-backend-1 bash
```

```bash
cd sites
```

```bash
vim common_site_config.json
```

Add this line:

```json
"host_name": "http://127.0.0.1:8000",
```

It should look like this:

```json
{
 "db_host": "db",
 "db_port": 3306,
 "host_name": "http://127.0.0.1:8000",
 "redis_cache": "redis://redis-cache:6379",
 "redis_queue": "redis://redis-queue:6379",
 "redis_socketio": "redis://redis-queue:6379",
 "socketio_port": 9000
}

Save and exit

```
