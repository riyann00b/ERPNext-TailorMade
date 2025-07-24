# ERPNext-TailorMade: A Guide to Custom Docker Installation

A comprehensive guide to installing a full ERPNext stack with Docker, custom apps, and NGINX on a local machine. This document is based on a production setup for a women's clothing store and includes post-installation workarounds, error fixes, and setup notes for enabling HTTPS with a self-signed certificate for Google Integration.

> ### Acknowledgements
> This guide is heavily inspired by Atul Bhatt's YouTube video: [Frappe Docker Custom Apps Installation using Quick Build Image | 2025](https://www.youtube.com/watch?v=FWGWKC_rZeI).
>
> The official Frappe Docker documentation is the primary source. Always check for updates there before following this guide: [Official Frappe Docker Documentation](https://github.com/frappe/frappe_docker/tree/main).

### Prerequisites

*   **Docker** installed and running.
*   **Docker Compose** installed.
*   **Git** installed.

---

## Part 1: Building the Custom Docker Image

### Step 1: Clone the Frappe Docker Repository

First, clone the official `frappe_docker` repository from GitHub to get the necessary build files.

```bash
git clone https://github.com/frappe/frappe_docker
cd frappe_docker
```

### Step 2: Configure Custom Applications via `apps.json`

Navigate into the `development` directory and create a file that lists the custom Frappe apps you want to install.

```bash
cd development
```

Create your own `apps.json` file. You can remove the example file first.

```bash
# Optional: remove the example file
rm -f apps-example.json

# Create and edit the new file using nano or your favorite editor
nano apps.json
```

**File Content (`apps.json`)**

This file is a JSON array of objects, where each object specifies a git repository and a branch for an app.

**Example `apps.json`:**

```json
[
  {
    "url": "https://github.com/frappe/erpnext",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/payments",
    "branch": "version-15"
  },
  {
    "url": "https://{{PAT}}@git.example.com/project/repository.git",
    "branch": "main"
  }
]
```

**Production `apps.json` used in this guide:**

```json
[
  {
    "url": "https://github.com/frappe/erpnext",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/hrms.git",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/payments",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/crm.git",
    "branch": "main"
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
    "url": "https://github.com/frappe/insights.git",
    "branch": "version-3"
  },
  {
    "url": "https://github.com/frappe/print_designer.git",
    "branch": "main"
  },
  {
    "url": "https://github.com/frappe/helpdesk.git",
    "branch": "main"
  }
]
```

> **Important Notes:**
>
> *   For private repositories, the `url` must be an HTTP(S) git URL containing a Personal Access Token (PAT), e.g., `https://{{PAT}}@github.com/project/repository.git`.
> *   You must manually add dependencies in `apps.json`. For example, `hrms` requires `erpnext` to be in the list.

### Step 3: Encode `apps.json` to Base64

The build process requires the `apps.json` content to be passed as a Base64 encoded environment variable.

```bash
# Make sure you are in the frappe_docker/development directory
export APPS_JSON_BASE64=$(base64 -w 0 apps.json)
```

To validate the string, you can decode it back and check the contents.

```bash
# Decode the variable into a test file
echo -n ${APPS_JSON_BASE64} | base64 -d > apps-test-output.json

# Display the decoded content and validate the original JSON syntax (requires jq)
cat apps-test-output.json
jq empty apps.json
```

### Step 4: Build the Custom Docker Image

#### Choosing a Build Method

**1. Quick Build (Recommended)**

This method uses pre-built `frappe/base` and `frappe/build` image layers, which significantly speeds up build time. It uses the `images/layered/Containerfile`.

```bash
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=your-docker-username/frappe-custom:latest \
  --file=images/layered/Containerfile .
```

**2. Custom Build (Advanced)**

This method builds the base layers from scratch, allowing you to customize runtime versions for Python and Node.js. It takes longer and uses the `images/custom/Containerfile`.

```bash
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=PYTHON_VERSION=3.11.9 \
  --build-arg=NODE_VERSION=20.19.2 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=your-docker-username/frappe-custom:latest \
  --file=images/custom/Containerfile .
```

#### Pushing the Image to a Registry

Log in to your container registry (e.g., Docker Hub), and push the image.

```bash
# Log in to Docker Hub
docker login

# Push the image
docker push your-docker-username/frappe-custom:latest
```

---

## Part 2: Configuring the Local Environment for HTTPS

To prepare for Google Integration, we will set up NGINX as a reverse proxy with a self-signed SSL certificate.

### Step 5: Generate the Self-Signed SSL Certificate

This certificate will encrypt traffic between your browser and NGINX.

```bash
# From your root `frappe_docker` directory, create a folder for the certificate
mkdir -p ssl

# Generate the certificate and key
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout ssl/erpnext-selfsigned.key \
  -out ssl/erpnext-selfsigned.crt
```

> **IMPORTANT:** When prompted, the only critical field is the **Common Name**. You **must** enter `localhost`. You can leave the other fields blank.

### Step 6: Create the NGINX Configuration

This file instructs the reverse proxy on how to handle traffic.

```bash
# Create a directory for the Nginx configuration
mkdir nginx

# Create the configuration file inside the new directory
nano nginx/nginx.conf
```

**File Content (`nginx/nginx.conf`):**

```nginx
server {
    listen 80;
    server_name localhost;
    # Redirect all HTTP traffic to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name localhost;

    # SSL Certificate files
    ssl_certificate /etc/nginx/ssl/erpnext-selfsigned.crt;
    ssl_certificate_key /etc/nginx/ssl/erpnext-selfsigned.key;

    # Proxy settings
    location / {
        proxy_pass http://frontend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## Part 3: Docker Compose Configuration and Launch

### Step 7: Create the Final `pwd.yml` Docker Compose File

This master file defines all your services, including the NGINX reverse proxy. **Delete all content** in your existing `pwd.yml` and **replace it** with the following:

```yaml
# In frappe_docker/pwd.yml

version: "3.8"

services:
  backend:
    image: your-docker-username/frappe-custom:latest # Use your custom image
    platform: linux/amd64 # For Apple M1/M2 compatibility
    deploy:
      restart_policy: { condition: on-failure }
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs
    environment:
      - DB_HOST=db
      - DB_PORT=3306
      - MARIADB_ROOT_PASSWORD=admin
    networks:
      - frappe_network

  configurator:
    image: your-docker-username/frappe-custom:latest # Use your custom image
    platform: linux/amd64
    deploy:
      restart_policy: { condition: none }
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs
    entrypoint:
      - bash
      - -c
      - >
        ls -1 apps > sites/apps.txt;
        bench set-config -g db_host $$DB_HOST;
        bench set-config -gp db_port $$DB_PORT;
        bench set-config -g redis_cache "redis://$$REDIS_CACHE";
        bench set-config -g redis_queue "redis://$$REDIS_QUEUE";
        bench set-config -g redis_socketio "redis://$$REDIS_QUEUE";
        bench set-config -gp socketio_port $$SOCKETIO_PORT;
    environment:
      - DB_HOST=db
      - DB_PORT=3306
      - REDIS_CACHE=redis-cache:6379
      - REDIS_QUEUE=redis-queue:6379
      - SOCKETIO_PORT=9000
    networks:
      - frappe_network

  create-site:
    image: your-docker-username/frappe-custom:latest # Use your custom image
    platform: linux/amd64
    deploy:
      restart_policy: { condition: none }
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs
    entrypoint:
      - bash
      - -c
      - >
        wait-for-it -t 120 db:3306;
        wait-for-it -t 120 redis-cache:6379;
        wait-for-it -t 120 redis-queue:6379;
        export start=`date +%s`;
        until [[ -n `grep -hs ^ sites/common_site_config.json | jq -r ".db_host // empty"` ]] && \
          [[ -n `grep -hs ^ sites/common_site_config.json | jq -r ".redis_cache // empty"` ]] && \
          [[ -n `grep -hs ^ sites/common_site_config.json | jq -r ".redis_queue // empty"` ]];
        do
          echo "Waiting for sites/common_site_config.json to be created...";
          sleep 5;
          if (( `date +%s`-start > 120 )); then echo "Timed out waiting for common_site_config.json"; exit 1; fi;
        done;
        echo "common_site_config.json found. Creating new site...";
        bench new-site --no-mariadb-socket --mariadb-root-username=root --mariadb-root-password=admin --admin-password=admin --install-app erpnext --install-app hrms --install-app payments --install-app crm --install-app ecommerce_integrations --install-app india-compliance --install-app insights --install-app print_designer --install-app helpdesk --set-default frontend;
    networks:
      - frappe_network

  db:
    image: mariadb:10.6
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "--password=admin"]
      interval: 1s
      retries: 20
    deploy:
      restart_policy: { condition: on-failure }
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
      - --skip-character-set-client-handshake
      - --skip-innodb-read-only-compressed
    environment:
      - MARIADB_ROOT_PASSWORD=admin
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - frappe_network

  frontend:
    image: your-docker-username/frappe-custom:latest # Use your custom image
    platform: linux/amd64
    deploy:
      restart_policy: { condition: on-failure }
    command:
      - nginx-entrypoint.sh
    environment:
      - BACKEND=backend:8000
      - FRAPPE_SITE_NAME_HEADER=frontend
      - SOCKETIO=websocket:9000
      - UPSTREAM_REAL_IP_ADDRESS=127.0.0.1
      - UPSTREAM_REAL_IP_HEADER=X-Forwarded-For
      - UPSTREAM_REAL_IP_RECURSIVE=off
      - PROXY_READ_TIMEOUT=120
      - CLIENT_MAX_BODY_SIZE=50m
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs
    depends_on:
      - websocket
      - backend
    networks:
      - frappe_network

  queue-long:
    image: your-docker-username/frappe-custom:latest # Use your custom image
    platform: linux/amd64
    deploy:
      restart_policy: { condition: on-failure }
    command: ["bench", "worker", "--queue", "long,default,short"]
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs
    networks:
      - frappe_network

  queue-short:
    image: your-docker-username/frappe-custom:latest # Use your custom image
    platform: linux/amd64
    deploy:
      restart_policy: { condition: on-failure }
    command: ["bench", "worker", "--queue", "short,default"]
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs
    networks:
      - frappe_network

  scheduler:
    image: your-docker-username/frappe-custom:latest # Use your custom image
    platform: linux/amd64
    deploy:
      restart_policy: { condition: on-failure }
    command: ["bench", "schedule"]
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs
    networks:
      - frappe_network

  websocket:
    image: your-docker-username/frappe-custom:latest # Use your custom image
    platform: linux/amd64
    deploy:
      restart_policy: { condition: on-failure }
    command: ["node", "/home/frappe/frappe-bench/apps/frappe/socketio.js"]
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs
    networks:
      - frappe_network

  redis-queue:
    image: redis:6.2-alpine
    deploy:
      restart_policy: { condition: on-failure }
    volumes:
      - redis-queue-data:/data
    networks:
      - frappe_network

  redis-cache:
    image: redis:6.2-alpine
    deploy:
      restart_policy: { condition: on-failure }
    networks:
      - frappe_network

  # New NGINX Reverse Proxy Service
  nginx:
    image: nginx:latest
    platform: linux/amd64
    deploy:
      restart_policy: { condition: on-failure }
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - frontend
    networks:
      - frappe_network

volumes:
  db-data:
  redis-queue-data:
  sites:
  logs:

networks:
  frappe_network:
    driver: bridge
```

### Summary of Changes

The original `pwd.yml` is a standard setup for a basic ERPNext installation. The modified version is tailored for a more advanced development environment with three primary goals:

1.  **Use Custom Apps:** The `image` for all Frappe/ERPNext services is changed from the official `frappe/erpnext` to a custom-built image (`riyann00b/frappe-custom:latest`). This custom image contains additional applications not included in the standard build.
2.  **Enable HTTPS with a Reverse Proxy:** A new `nginx` service is added to act as a reverse proxy. This allows the use of a self-signed SSL certificate, making the local environment accessible via `https://localhost`. This is often a prerequisite for testing third-party integrations (like Google or Stripe).
3.  **Ensure Compatibility:** The `platform: linux/amd64` directive is added to ensure the images run correctly on ARM-based machines like Apple M1/M2 Macs via Rosetta emulation.

---

### Detailed `diff` Comparison

Here is the line-by-line comparison of the `pwd.yml` file.

````diff
# No changes to the version
version: "3"

services:
  # --- Backend Service ---
  backend:
-   image: frappe/erpnext:v15.70.2
+   # MODIFICATION: Use the custom-built image that includes all our apps.
+   image: riyann00b/frappe-custom:latest
+   # MODIFICATION: Add platform for Apple M1/M2 compatibility.
+   platform: linux/amd64
# ... no other changes to backend ...
    networks:
      - frappe_network

  # --- Configurator Service ---
  configurator:
-   image: frappe/erpnext:v15.70.2
+   # MODIFICATION: Use the custom-built image.
+   image: riyann00b/frappe-custom:latest
+   # MODIFICATION: Add platform for Apple M1/M2 compatibility.
+   platform: linux/amd64
# ... no other changes to configurator ...
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs

  # --- Create-Site Service ---
  create-site:
-   image: frappe/erpnext:v15.70.2
+   # MODIFICATION: Use the custom-built image.
+   image: riyann00b/frappe-custom:latest
+   # MODIFICATION: Add platform for Apple M1/M2 compatibility.
+   platform: linux/amd64
# ...
    command:
      - >
# ... wait-for-it and until blocks are identical ...
        echo "sites/common_site_config.json found";
-       bench new-site --mariadb-user-host-login-scope='%' --admin-password=admin --db-root-username=root --db-root-password=admin --install-app erpnext --set-default frontend;
+       # MODIFICATION: The site creation command is updated to install all custom apps from the start.
+       bench new-site --mariadb-user-host-login-scope='%' --admin-password=admin --db-root-username=root --db-root-password=admin --install-app erpnext --install-app payments --install-app ecommerce_integrations --install-app india_compliance --install-app insights --install-app print_designer --install-app helpdesk  --set-default frontend;

  # --- DB Service ---
  # No changes to the db service. It remains the same.
  db:
    image: mariadb:10.6
# ...

  # --- Frontend Service ---
  frontend:
-   image: frappe/erpnext:v15.70.2
+   # MODIFICATION: Use the custom-built image.
+   image: riyann00b/frappe-custom:latest
+   # MODIFICATION: Add platform for Apple M1/M2 compatibility.
+   platform: linux/amd64
    networks:
      - frappe_network
    depends_on:
      - websocket
+     # MODIFICATION: Add dependency on backend to ensure correct startup order.
+     - backend
# ...
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs
-   ports:
-     - "8080:8080"
+   # MODIFICATION: The frontend port is removed because the new NGINX service
+   # will now manage external traffic on ports 80 and 443.

  # --- Worker & Scheduler Services ---
  # All worker, scheduler, and websocket services are updated to use the custom image and platform.
  queue-long:
-   image: frappe/erpnext:v15.70.2
+   image: riyann00b/frappe-custom:latest
+   platform: linux/amd64
# ...

  queue-short:
-   image: frappe/erpnext:v15.70.2
+   image: riyann00b/frappe-custom:latest
+   platform: linux/amd64
# ...

  scheduler:
-   image: frappe/erpnext:v15.70.2
+   image: riyann00b/frappe-custom:latest
+   platform: linux/amd64
# ...

  websocket:
-   image: frappe/erpnext:v15.70.2
+   image: riyann00b/frappe-custom:latest
+   platform: linux/amd64
# ...

  # --- Redis Services ---
  # No changes to redis-queue or redis-cache. They remain the same.

+ # --- NGINX Reverse Proxy Service ---
+ # NEW SERVICE: An NGINX reverse proxy is added to handle SSL termination (HTTPS)
+ # and forward traffic to the frontend service. This is the key to enabling https://localhost.
+ nginx:
+   image: nginx:latest
+   platform: linux/amd64
+   deploy:
+     restart_policy:
+       condition: on-failure
+   ports:
+     - "80:80"
+     - "443:443"
+   volumes:
+     - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
+     - ./ssl:/etc/nginx/ssl:ro
+   depends_on:
+     - frontend
+   networks:
+     - frappe_network

# --- Volumes and Networks ---
# No changes to the final volumes and networks sections.
volumes:
  db-data:
  redis-queue-data:
  sites:
  logs:

networks:
  frappe_network:
    driver: bridge

````

### **Optional Step: Configuring Host Access for Active Development**

While the main guide focuses on deploying a complete ERPNext application using `pwd.yml`, the `frappe_docker` repository also includes a separate setup for **active development** using the `development/docker-compose.yml` file. This setup is ideal for developers who are actively building or modifying custom apps.

This optional configuration allows the Frappe development container to communicate with services running directly on your computer's `localhost`. This is useful if you are testing webhooks or need to connect to another local service (like a local n8n instance) from within the container.

#### How It Works

This is achieved by adding an `extra_hosts` block to the `frappe` service in the `development/docker-compose.yml` file. This block adds a custom entry to the container's internal hosts file.

The recommended approach is to use Docker's special DNS name, `host.docker.internal`, which reliably points to your host machine's IP address across different platforms.

#### Configuration

1.  Open the development-specific docker compose file for editing:
    ```bash
    # Make sure you are in the frappe_docker directory
    nano development/docker-compose.yml
    ```

2.  Locate the `frappe` service and add the `extra_hosts` block as shown below.

    **Example: Adding Host Aliases to `development/docker-compose.yml`**

    ```yaml
    # ... (other services like mariadb, redis) ...

    frappe:
      image: docker.io/frappe/bench:latest
      platform: linux/amd64
      command: sleep infinity
      environment:
        - SHELL=/bin/bash
      volumes:
        - ..:/workspace:cached
        # Enable if you require git cloning
        - ${HOME}/.ssh:/home/frappe/.ssh
      working_dir: /workspace/development
      ports:
        - 8000-8005:8000-8005
        - 9000-9005:9000-9005
      # ADD THIS BLOCK to allow the container to connect back to your host machine
      # using custom names.
      extra_hosts:
        - "app1.localhost:host.docker.internal"
        - "app2.localhost:host.docker.internal"

    # ... (rest of the file) ...
    ```

After adding this block and starting the development environment with `docker compose -f development/docker-compose.yml up -d`, any application running inside the `frappe` container will be able to reach a service on your host's `localhost` by using the address `http://app1.localhost`.

### Step 8: Launch the Secure Stack and Finalize Setup

1.  **Launch all services:**
    ```bash
    docker compose -f pwd.yml up -d
    ```

2.  **Access your site.** Open your browser and navigate to **`https://localhost`**. You will need to bypass the browser's security warning for your self-signed certificate.

3.  **Set the `host_name`.** This final step ensures ERPNext generates correct HTTPS links.
    > **Warning:** Do NOT manually edit `common_site_config.json`. A syntax error can crash your site. Always use the `bench` command.

    ```bash
    # Find the exact container name for the backend service
    docker compose -f pwd.yml ps

    # Use the name to run the command (replace 'frappe_docker-backend-1' if yours differs)
    docker exec frappe_docker-backend-1 bench set-config -g host_name https://localhost
    ```

4.  **Restart the stack** for the change to apply everywhere.
    ```bash
    docker compose -f pwd.yml restart
    ```
---

## Phase 1: Google Cloud Platform Configuration

The first and most critical phase is to create OAuth 2.0 credentials in the Google Cloud Platform. These credentials allow your ERPNext instance to communicate with Google's APIs securely.

### Step 1: Create a New Google Cloud Project

1.  **Navigate to the Google API Console:** Open the [Google API Console](https://console.developers.google.com/).
2.  **Create a Project:** If you don't have a project, click the project dropdown (top-left) and select "**New Project**".
    *   **Project Name:** Give it a descriptive name, like "ERPNext Integration".
    *   Click "**Create**".

### Step 2: Enable the Necessary APIs

For each Google service you want to integrate, you must enable its corresponding API.

1.  From your project's Dashboard, click on "**+ ENABLE APIS AND SERVICES**".
2.  Search for and enable the APIs you need. For a full integration, you will need:
    *   **Google Drive API** (for backups)
    *   **Google Calendar API** (for calendar sync)
    *   **People API** (for Google Contacts sync)
3.  Click on each API and then click the "**Enable**" button.

### Step 3: Configure the OAuth Consent Screen

This screen is what your users will see when they are asked to grant permission to your ERPNext application.

1.  In the left-hand navigation menu, go to "**OAuth consent screen**".
2.  **User Type:** Choose "**External**" and click "**Create**". This is the standard choice for most applications.
3.  **App Information:**
    *   **App name:** Enter a name for your application, e.g., "ERPNext Local Dev".
    *   **User support email:** Select your email address.
    *   **App logo:** (Optional) You can upload a logo.
4.  **App Domain:** This section is crucial for security.
    *   As noted: *To protect you and your users, Google only allows apps using OAuth to use Authorized Domains. The following information will be shown to your users on the consent screen.*
    *   **Application home page:** For a local setup, enter `https://localhost/app/home`. If you have a purchased domain, you would use that instead (e.g., `https://your-erp-domain.com/app/home`).
    *   **Application privacy policy link / terms of service link:** For local development, you can use `https://localhost` for these. For a production app, you must provide links to your actual policies.
5.  **Authorized domains:**
    *   As noted: *When a domain is used on the consent screen or in an OAuth client’s configuration, it must be pre-registered here.*
    *   Click "**+ ADD DOMAIN**" and enter `localhost`.
    *   If you are using a purchased domain for a live site, you would add that domain here (e.g., `your-erp-domain.com`).
6.  **Developer contact information:** Enter your email address.
7.  Click "**SAVE AND CONTINUE**".
8.  **Scopes & Test Users:** For now, you can skip the "Scopes" and "Test Users" sections by clicking "**SAVE AND CONTINUE**" and then "**BACK TO DASHBOARD**".

### Step 4: Create OAuth 2.0 Client ID

This is where you will define the specific URIs your ERPNext application is allowed to use for authentication and get your final credentials.

1.  In the left-hand navigation menu, go to "**Credentials**".
2.  Click on "**+ CREATE CREDENTIALS**" and select "**OAuth client ID**".
3.  **Configure the Client ID:**
    *   **Application type:** Select "**Web application**".
    *   **Name:** Give it a descriptive name, like "ERPNext Localhost Client".
4.  **Authorized JavaScript origins:** This is for requests originating from a browser.
    *   Click "**+ ADD URI**".
    *   Enter `https://localhost`.
    *   *(For a production environment, you would add your domain, e.g., `https://your-erp-domain.com`)*.
5.  **Authorized redirect URIs:** This is the specific endpoint in ERPNext that Google will send the authentication response to.
    *   Click "**+ ADD URI**".
    *   Enter the following exact URI for a standard Frappe/ERPNext setup:
        ```
        https://localhost/api/method/frappe.integrations.google_oauth.callback
        ```
    *   *(For a production environment, replace `localhost` with your domain: `https://your-erp-domain.com/api/method/frappe.integrations.google_oauth.callback`)*.
6.  Click "**CREATE**".

### Step 5: Get Your Credentials

A pop-up window will now appear titled "**OAuth client created**". It will display your `Client ID` and `Client secret`.

*   **Your Client ID:** `(a long string of characters)`
*   **Your Client Secret:** `(a shorter, secret string of characters)`

**Copy both of these values and keep them secure.** You will need them for the next phase.

> **Important Note:** As the console mentions, *it may take 5 minutes to a few hours for these new settings to take effect* on Google's side. If you get an error immediately after setup, wait a while and try again.

## Phase 2: ERPNext Configuration

Now, with the credentials from Google, you can configure ERPNext to connect to your newly created app.

1.  **Navigate to Google Settings in ERPNext:** Log in to your ERPNext instance and use the awesome bar to search for "**Google Settings**".
2.  **Enter Credentials:**
    *   In the "Google Settings" page, find the fields for "**Client ID**" and "**Client Secret**".
    *   Paste the values you copied from the Google API Console in the previous phase.
3.  **Enable Google Integration:** Check the box that says "**Enable**".
4.  **Save:** Click the "**Save**" button at the top of the page.

Your general Google integration is now configured. You can proceed to authorize and use individual services like Google Drive, Calendar, and Contacts. The authorization process for each service will now use the credentials you just saved and will redirect you to the Google consent screen you configured.
---
Your local ERPNext instance is now fully configured for development and ready for Google Integration.

---

## Part 4: Post-Installation & Troubleshooting

### Fixing `wkhtmltopdf` Errors

If you encounter a `ConnectionRefusedError` when generating PDFs, the `host_name` might be incorrect. The fix in Step 8 (setting `host_name` to `https://localhost`) should resolve this for the NGINX setup.

If issues persist, verify dependencies inside the container and consider switching to Chrome for PDF generation.

1.  **Enter the backend container:**
    ```bash
    docker exec -it frappe_docker-backend-1 bash
    ```

2.  **Verify `wkhtmltopdf`:** Check that the version is compiled "with patched qt".
    ```bash
    wkhtmltopdf --version
    ```
    If it's missing, you may need a custom image with it pre-installed. Also ensure `xvfb-run` is available (`xvfb-run --help`).

3.  **Exit the container:**
    ```bash
    exit
    ```

#### Alternative Workaround: Use Chrome PDF Engine

1.  In your ERPNext desk, search for and go to **Print Format List**.
2.  Find the format you are using (e.g., "Sales Invoice").
3.  From the menu, duplicate the format.
4.  In the duplicated format, change the **Print Engine** from `Wkhtmltopdf` to **Chrome**.
5.  Save and set this new print format as the default.

### Troubleshooting NGINX Mount Errors

**Problem:** You may encounter an error during `docker compose up` that says:
`Error response from daemon: ... error mounting ".../nginx/nginx.conf" ... not a directory`

**Cause:** This happens when Docker expects `./nginx` to be a directory on your host machine, but it is a file instead. This prevents Docker from finding `nginx.conf` *inside* it.

**Solution:** Forcefully delete the incorrect entry and create the proper directory structure.

1.  **Stop all services:**
    ```bash
    docker compose -f pwd.yml down
    ```

2.  **Forcefully delete the `nginx` file/directory from your host:**
    ```bash
    rm -rf nginx
    ```

3.  **Verify it is gone.** This command should now fail:
    ```bash
    ls -l nginx
    ```

4.  **Re-create the directory and the configuration file correctly:**
    ```bash
    mkdir nginx
    nano nginx/nginx.conf
    ```

5.  Paste the NGINX configuration from **Step 6** into the file, then save and exit.

6.  **Relaunch the stack.** Using `--force-recreate` ensures the containers are built fresh with the corrected volume mount.
    ```bash
    docker compose -f pwd.yml up -d --force-recreate
    ```
