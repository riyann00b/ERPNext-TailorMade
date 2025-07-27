# 🚀 ERPNext-TailorMade: A Guide to Custom Docker Installation

A comprehensive guide to installing a full ERPNext stack with Docker, including custom apps and an NGINX reverse proxy for local HTTPS. This document details a production-inspired setup, including post-installation workarounds and instructions for enabling Google Integration with a self-signed certificate.

> ### 🙏 Acknowledgements
> *   This guide is heavily inspired by Atul Bhatt's YouTube video: [Frappe Docker Custom Apps Installation using Quick Build Image | 2025](https://www.youtube.com/watch?v=FWGWKC_rZeI).
> *   The official Frappe Docker documentation is the primary source. Always check for updates there before following this guide: [Official Frappe Docker Documentation](https://github.com/frappe/frappe_docker/tree/main).

---

## 📋 Table of Contents

- [Prerequisites](#Prerequisites)
- Part 1: Building the Custom Docker Image
- Part 2: Configuring the Local Environment for HTTPS
- Part 3: Docker Compose Configuration and Launch
- Part 4: Configuring Google Integration
- Part 5: Post-Installation & Troubleshooting
- Part 6: Setting Up Products and Invoices for a Women's Clothing Store
- Part 7: Integrating n8n for Automation
- Part 8: Domain and SSL Setup for Production
- Bench CLI Cheatsheet
- [Resources](#resources)

---

### ✅ Prerequisites

Ensure the following software is installed and running:
- **Docker**
- **Docker Compose**
- **Git**
- **OpenSSL** (for generating self-signed SSL certificates)
- **Optional**: `jq` (for JSON validation), `nano` or `VS Code` (for editing files)

---

## Part 1: Building the Custom Docker Image

This section covers how to build a custom Docker image that includes ERPNext and any additional Frappe apps you require.

### Step 1: Clone the Frappe Docker Repository
Clone the official `frappe_docker` repository and navigate into the `development` directory.

```bash
git clone https://github.com/frappe/frappe_docker
cd frappe_docker/development
```

### Step 2: Configure Custom Applications via `apps.json`
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
    "url": "https://github.com/frappe/payments",
    "branch": "version-15"
  },
  {
    "url": "https://{{ PAT }}@git.example.com/project/repository.git",
    "branch": "main"
  }
]
```

**`apps.json` Production File Used in this Guide:**
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
> #### ❗ Important Notes:
> - **Dependencies:** You must manually add app dependencies. For example, `hrms` requires `erpnext` to be in the list.
> - **Private Repos:** For private repositories, use an HTTPS git URL with a Personal Access Token (PAT): `https://{{PAT}}@github.com/your-org/your-private-app.git`.

### Step 3: Encode `apps.json` to Base64
The Docker build process requires the `apps.json` content to be passed as a Base64 encoded string.

```bash
# Ensure you are in the frappe_docker/development directory
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

### Step 4: Build and Push the Custom Docker Image
This is a multi-stage process to build, tag, and push your image.

#### 🏗️ A. Build the Image
Navigate back to the root `frappe_docker` directory.
```bash
cd ..
```
Run the build command using the **Quick Build** method for faster builds.

**Quick Build Command Used in this Guide:**
example: 
> Change the tag to your docker username and tag the image as latest
```docker
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=ghcr.io/user/repo/custom:1.0.0 \
  --file=images/layered/Containerfile .
```

Production:
```bash
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=riyann00b/frappe-custom:latest \
  --file=images/layered/Containerfile .
```
> ☕ Sit back, relax, and have a coffee. This will take a while.

#### 🏷️ B. Custom Build Image (Full Image, Full Control)
This method builds the base and build layer every time, allowing customization of Python and NodeJS runtime versions. It takes more time to build.

**Example:**
```bash
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=PYTHON_VERSION=3.11.9 \
  --build-arg=NODE_VERSION=20.19.2 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=ghcr.io/user/repo/custom:1.0.0 \ #change it to  --tag=Your-Docker-username/frappe-custom:latest 
  --file=images/custom/Containerfile .
```
OR Manually Tag the image:
```bash
docker tag ghcr.io/user/repo/custom:1.0.0 riyann00b/frappe-custom:latest
```

Verify the new image:
```bash
docker images
```
Optionally, remove the old intermediate tag:
```bash
docker rmi ghcr.io/user/repo/custom:1.0.0
```

#### 🚀 C. Push the Image to a Registry
Log in to your container registry and push the image.

```bash
# Log in to Docker Hub (or your chosen registry)
docker login

# Push the final image
docker push riyann00b/frappe-custom:latest
```

---

## Part 2: Configuring the Local Environment for HTTPS

Set up NGINX as a reverse proxy with a self-signed SSL certificate to enable `https://` access on `localhost`.

### Step 5: Generate a Self-Signed SSL Certificate
From your root `frappe_docker` directory, create a directory for the certificate and generate the files.

```bash
mkdir -p ssl

openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout ssl/erpnext-selfsigned.key \
  -out ssl/erpnext-selfsigned.crt
```
> **❗ IMPORTANT:** When prompted, the only critical field is the **Common Name**. You **must** enter `localhost`. All other fields can be left blank by pressing Enter.

### Step 6: Create the NGINX Configuration File
Create a directory and a configuration file for NGINX.

```bash
mkdir nginx
nano nginx/nginx.conf
```

Paste the following content into `nginx/nginx.conf`:
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

This section describes the master `pwd.yml` file that orchestrates all services.

### Step 7: Create the Final `pwd.yml` Docker Compose File
Navigate to the `frappe_docker` directory if not already there:

```bash
cd /frappe_docker
```

Edit the `pwd.yml` file with your preferred text editor:

```bash
# Using nano
sudo nano pwd.yml

# Or using VS Code
code pwd.yml
```

Replace all instances of the default image (`frappe/erpnext:v15.xx.x`) with your custom image:

```bash
riyann00b/frappe-custom:latest
```

Add `platform: linux/amd64` to all services in the `pwd.yml` file to ensure compatibility (e.g., for Apple M1/M2 systems).

Update the `create-site` service's command to include your custom apps for installation. Example modification in `pwd.yml`:

```yaml
services:
  create-site:
    command:
      - >
        bench new-site --mariadb-user-host-login-scope='%' --admin-password=admin --db-root-username=root --db-root-password=admin --install-app erpnext --install-app hrms --install-app payments --install-app crm --install-app ecommerce_integrations --install-app india_compliance --install-app insights --install-app print_designer --install-app helpdesk --set-default frontend;
```

copy only the apps
```yaml
--install-app hrms --install-app payments --install-app crm --install-app ecommerce_integrations --install-app india_compliance --install-app insights --install-app print_designer --install-app helpdesk
```

**Optional**: Add `extra_hosts` to the `/frappe_docker/devcontainer-example/docker-compose.yml` service for custom domain resolution:
```yaml
services:
  frappe:
    extra_hosts:
      app1.localhost: 172.17.0.1
      app2.localhost: 172.17.0.1
```

The complete `pwd.yml` file used in this guide is available at:
*   [**`pwd.yml` on GitHub**](https://github.com/riyann00b/ERPNext-TailorMade/blob/main/pwd.yml)

The key changes from the default file are summarized below. The `diff` view provides a clear line-by-line comparison of what was added or changed.

<details>
<summary><strong>Click to expand and view the `diff` of changes in `pwd.yml`</strong></summary>

```diff
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

  # --- Configurator Service ---
  configurator:
-   image: frappe/erpnext:v15.70.2
+   image: riyann00b/frappe-custom:latest
+   platform: linux/amd64
# ... no other changes to configurator ...

  # --- Create-Site Service ---
  create-site:
-   image: frappe/erpnext:v15.70.2
+   image: riyann00b/frappe-custom:latest
+   platform: linux/amd64
# ...
    command:
      - >
# ... wait-for-it block is identical ...
-       bench new-site --mariadb-user-host-login-scope='%' --admin-password=admin --db-root-username=root --db-root-password=admin --install-app erpnext --set-default frontend;
+       # MODIFICATION: The site creation command is updated to install all custom apps from the start.
+       bench new-site --mariadb-user-host-login-scope='%' --admin-password=admin --db-root-username=root --db-root-password=admin --install-app erpnext --install-app payments --install-app ecommerce_integrations --install-app india_compliance --install-app insights --install-app print_designer --install-app helpdesk  --set-default frontend;

  # --- Frontend Service ---
  frontend:
-   image: frappe/erpnext:v15.70.2
+   image: riyann00b/frappe-custom:latest
+   platform: linux/amd64
    depends_on:
      - websocket
+     # MODIFICATION: Add dependency on backend to ensure correct startup order.
+     - backend
-   ports:
-     - "8080:8080"
+   # MODIFICATION: The frontend port is removed because the new NGINX service
+   # will now manage external traffic on ports 80 and 443.

  # --- All other Frappe services (queue-long, queue-short, scheduler, websocket) ---
  # are similarly updated to use the custom image and platform.
-   image: frappe/erpnext:v15.70.2
+   image: riyann00b/frappe-custom:latest
+   platform: linux/amd64

+ # --- NGINX Reverse Proxy Service (NEW) ---
+ # NEW SERVICE: An NGINX reverse proxy is added to handle SSL termination (HTTPS).
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
```

</details>

### Step 8: Launch the Stack and Finalize Setup

1. **Launch all services:**
   From the root `frappe_docker` directory, run:
   ```bash
   docker compose -f pwd.yml up -d
   ```
   
2. **Monitor site creation**:
   ```bash
   docker logs frappe_docker-create-site-1 -f
   ```
   
3. **Set the `host_name` for correct URL generation:**
   This ensures ERPNext generates correct `https://` links.
   ```bash
   # Find the backend container's name
   docker ps --filter "name=backend"

   # Replace 'frappe_docker-backend-1' with your actual container name
   docker exec frappe_docker-backend-1 bench set-config -g host_name https://localhost
   ```
4. **Access your site:**
   Open your browser and navigate to **[https://localhost](https://localhost)**. Accept the security warning for the self-signed certificate.
   
5. **Restart the stack** for the change to take effect:
   ```bash
   docker compose -f pwd.yml restart
   ```

---

## Part 4: Configuring Google Integration

Generate credentials from Google Cloud and configure them in ERPNext for services like Drive, Calendar, and Gmail.

### Phase 1: Google Cloud Platform Configuration

#### Step 1: Create a New Google Cloud Project
1. Go to the [Google API Console](https://console.developers.google.com/).
2. From the project dropdown, select **New Project**.
3. Name it (e.g., "ERPNext Integration") and click **Create**.

#### Step 2: Enable the Necessary APIs
1. From the Dashboard, click **+ ENABLE APIS AND SERVICES**.
2. Enable the following APIs:
   - **Google Drive API**
   - **Google Calendar API**
   - **People API** (for Google Contacts)
   - **Gmail API**

#### Step 3: Configure the OAuth Consent Screen
1. Go to **OAuth consent screen** in the left menu.
2. Choose **External** User Type and click **Create**.
3. Fill in app information (App name, support email, etc.).
4. Under **Authorized domains**, add `localhost`.
5. Click **SAVE AND CONTINUE**.

#### Step 4: Create Credentials
**A. Create OAuth 2.0 Client ID:**
1. Go to **Credentials** -> **+ CREATE CREDENTIALS** -> **OAuth client ID**.
2. Set **Application type** to **Web application**.
3. Add `https://localhost` to **Authorized JavaScript origins**.
4. Add `https://localhost/api/method/frappe.integrations.google_oauth.callback` to **Authorized redirect URIs**.
5. Click **CREATE**.

**B. Create API Key:**
1. Go to **Credentials** -> **+ CREATE CREDENTIALS** -> **API Key**.
2. Copy the generated key.

#### Step 5: Copy Your Credentials
From the **Credentials** page, copy your **Client ID**, **Client Secret**, and **API Key**.

### Phase 2: ERPNext General Configuration
1. Log in to your ERPNext instance at `https://localhost`.
2. Search for **Google Settings**.
3. Check the **Enable** box.
4. Paste your **Client ID**, **Client Secret**, and **API Key**.
5. Click **Save**.

### Phase 3: Setting Up Specific Google Services in ERPNext

#### 📦 Google Drive Setup
1. In **Google Settings**, scroll to the "Google Drive" section.
2. Click **Authorize Google Drive Access** and complete the consent flow.
3. To enable backups, go to **Download Backups** and select "Google Drive" as the storage location.

#### 📅 Google Calendar Setup
1. In **Google Settings**, click **Authorize Google Calendar Access** and complete the consent flow.
2. Users must configure personal sync options in **My Settings** -> **Google Calendar**.

#### 👤 Google Contacts Setup
1. In **Google Settings**, click **Authorize Google Contacts Access** and complete the consent flow.
2. Click **Sync Google Contacts** to begin synchronization.

#### ✉️ Google Email (Gmail) Setup
1. **Generate a Google App Password**:
   - Go to your Google Account -> **Security**.
   - Enable **2-Step Verification**.
   - Select **App passwords**, generate a password for "Mail" on "Windows Computer", and copy the 16-character password.
2. **Configure Email Domain in ERPNext**:
   - Open the **Email Domain** list, create a new entry for `gmail.com`, and select the `Gmail` service.
3. **Configure Email Account in ERPNext**:
   - Open the **Email Account** list and create a new account.
   - Enter your Gmail address and select the `gmail.com` domain.
   - Paste the 16-character **App Password** into the **Password** field.
   - Enable incoming/outgoing mail and save.

---

## Part 5: Post-Installation & Troubleshooting

### 🔧 Fixing `wkhtmltopdf` PDF Errors
**Problem:** `ConnectionRefusedError` when generating PDFs.
- **Solution 1 (Primary Fix):** Ensure the `host_name` is set to `https://localhost` (see **Step 8**).
- **Solution 2 (Alternative):** Switch to the Chrome PDF engine in **Print Format List** by duplicating your format and setting **Print Engine** to **Chrome**.

### 🔧 Troubleshooting NGINX Mount Errors
**Problem:** Docker Compose fails with `error mounting ".../nginx/nginx.conf" ... not a directory`.
- **Cause:** Docker created a *file* named `nginx` instead of a directory.
- **Solution:**
  1. Stop all services: `docker compose -f pwd.yml down`
  2. Delete the incorrect entry: `rm -rf nginx`
  3. Verify it's gone: `ls -l nginx` (should fail)
  4. Re-create the directory and file (see **Step 6**).
  5. Relaunch the stack: `docker compose -f pwd.yml up -d --force-recreate`
