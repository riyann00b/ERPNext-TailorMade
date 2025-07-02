# ERPNext-TailorMade

## ERPNext Docker Installation & Setup Guide

A comprehensive guide to installing a full ERPNext stack with Docker, custom apps, and NGINX on a local machine. This document is based on a production setup for a women's clothing store and includes post-installation workarounds, error fixes, and setup notes.

### Acknowledgements

> This guide is heavily inspired by Atul Bhatt's YouTube video: [Frappe Docker Custom Apps Installation using Quick Build Image | 2025](https://www.youtube.com/watch?v=FWGWKC_rZeI).
>
> **Note:** The official Frappe Docker documentation is the primary source. Always check for updates there before following this guide.
> [Official Frappe Docker Documentation](https://github.com/frappe/frappe_docker/tree/main)

### Prerequisites

*   **Docker** installed and running.
*   **Docker Compose** installed.
*   **Docker Desktop** installed (recommended for easy management).

---

## Part 1: Docker Installation with Custom Apps

### Step 1: Clone the Frappe Docker Repository

First, clone the official `frappe_docker` repository from GitHub to get the necessary files.

```bash
git clone https://github.com/frappe/frappe_docker
```

### Step 2: Configure Custom Applications via `apps.json`

Navigate into the development directory to create a file that lists all the custom Frappe apps you want to install.

```bash
cd frappe_docker/development
```

Create your own `apps.json` file. You can remove the example file first.

```bash
# Optional: remove the example file
rm -f apps-example.json

# Create and edit the new file using nano or VS Code
nano apps.json
# or
code apps.json
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
    "url": "https://{{ PAT }}@git.example.com/project/repository.git",
    "branch": "main"
  }
]
```

**My Production `apps.json`:**
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
> *   For private repositories, the `url` must be an HTTP(S) git URL containing a Personal Access Token (PAT), e.g., `https://{{PAT}}@github.com/project/repository.git`.
> *   You must manually add dependencies in `apps.json`. For example, if you install `hrms`, you must also include `erpnext`.

### Step 3: Encode `apps.json` to Base64

The build process requires the `apps.json` content to be passed as a Base64 encoded environment variable.

```bash
# Make sure you are in the `frappe_docker/development` directory
export APPS_JSON_BASE64=$(base64 -w 0 apps.json)
```

**Validate the String**

To ensure the encoding was successful, you can decode it back and check the contents.

1.  Decode the variable into a test file:
    ```bash
    echo -n ${APPS_JSON_BASE64} | base64 -d > apps-test-output.json
    ```

2.  Display the decoded content:
    ```bash
    cat apps-test-output.json
    ```

3.  Validate the original JSON syntax (requires `jq`):
    ```bash
    jq empty apps.json
    ```

### Step 4: Build the Custom Docker Image

Now, navigate back to the root `frappe_docker` directory.

```bash
cd ..
```

#### Choosing a Build Method

There are two main methods to build your image.

**1. Quick Build (Recommended)**

My Production
```shell
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=riyann00b/frappe-custom:latest \
  --file=images/layered/Containerfile .
```
manual tagging

This method uses pre-built `frappe/base` and `frappe/build` image layers, which significantly speeds up build time. It uses the `images/layered/Containerfile`.

```shell
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=ghcr.io/user/repo/custom:1.0.0 \
  --file=images/layered/Containerfile .
```

**2. Custom Build (Advanced)**

This method builds the base layers from scratch every time, allowing you to customize runtime versions for Python and Node.js. It takes longer to build and uses the `images/custom/Containerfile`.

```shell
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=PYTHON_VERSION=3.11.9 \
  --build-arg=NODE_VERSION=20.19.2 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=ghcr.io/user/repo/custom:1.0.0 \
  --file=images/custom/Containerfile .
```

**Custom Build Arguments:**
*   `PYTHON_VERSION`: Specify the Python version (default: `3.11.6`).
*   `NODE_VERSION`: Specify the Node.js version (default: `20.19.2`).
*   `DEBIAN_BASE`: Specify the base Debian version (default: `bookworm`).
*   `WKHTMLTOPDF_VERSION`: Specify the `wkhtmltopdf` version (default: `0.12.6.1-3`).
*   `WKHTMLTOPDF_DISTRO`: Specify the distro for the Debian package (default: `bookworm`).

#### Tagging and Pushing the Image

1.  **Tag the Image:** After building, tag the image with a user-name that you can push to a registry like Docker Hub.

    ```bash
    # Example
    docker tag ghcr.io/user/repo/custom:1.0.0 docker_username/frappe-custom:latest
    
    # My Production Tag
    docker tag ghcr.io/user/repo/custom:1.0.0 riyann00b/frappe-custom:latest
    ```
    > *Sit back, relax, and have a coffee. The build will take a while.*

3.  **Verify and Push:**
    Log in to Docker Hub, verify the image exists, and push it to the registry.
    ```bash
    docker login
    docker images

    #example
    docker push docker_username/frappe-custom:latest

    #my productiom
    docker push riyann00b/frappe-custom:latest
    ```

### Step 5: Configure and Run Docker Compose

Edit the `pwd.yml` file to use your new custom image and to install the apps on the new site.

```bash
# Edit the file with your favorite editor
code pwd.yml
# or
sudo nano pwd.yml
```

Inside `pwd.yml`:

1.  **Replace Image:** Find all instances of `image: frappe/erpnext:v15...` and replace them with your custom image tag (`riyann00b/frappe-custom:latest`).
2.  **Add Platform (if needed):** For compatibility (e.g., on Apple M1/M2), add `platform: linux/amd64` to all services.
3.  **Install Custom Apps:** Find the `create-site` (80ish line) service and add your apps to the `command` section.

**My Production `create-site` command section:**
```yaml
bench new-site --mariadb-user-host-login-scope='%' --admin-password=admin --db-root-username=root --db-root-password=admin --install-app erpnext --install-app hrms --install-app payments --install-app crm --install-app ecommerce_integrations --install-app india_compliance --install-app insights --install-app print_designer --install-app helpdesk --set-default frontend;

```

#### Optional: Configure Host Aliases
To allow containers to connect to services running on the host machine's `localhost`, add `extra_hosts` to your Frappe service in `.devcontainer/docker-compose.yml`.

```yaml
  frappe:
    ...
    extra_hosts:
      app1.localhost: 172.17.0.1
      app2.localhost: 172.17.0.1
```

#### Launch the Stack

Once `pwd.yml` is saved, run the following command to start all services.

```bash
docker compose -f pwd.yml up -d
```

Follow the site creation logs to ensure all apps are installed correctly.

```bash
docker logs frappe_docker-create-site-1 -f
```

Once it's finished, you can access your ERPNext site at **[http://localhost:8080](http://localhost:8080)**.

---

## Part 2: Post-Installation & Workarounds

### Fixing `wkhtmltopdf` Error in Print Designer

**Problem:** You may encounter an error when generating PDFs: `OSError: wkhtmltopdf reported an error: Exit with code 1 due to network error: ConnectionRefusedError`

**Solution:** This is often a hostname resolution issue inside the container.

1.  **Access the Backend Container's Shell:**
    ```bash
    # Confirm 'frappe_docker-backend-1' is your backend container name with 'docker compose ps'
    docker exec -u 0 -it frappe_docker-backend-1 bash
    ```
2.  **Navigate to the Sites Directory:**
    ```bash
    cd /home/frappe/frappe-bench/sites/
    #or
    cd sites
    ```
3.  **Edit `common_site_config.json`:**
    ```bash
    nano common_site_config.json
    #or
    vi common_site_config.json
    ```
4.  **Add the `host_name` Key:** Add the following line to the JSON object. This tells `wkhtmltopdf` to use the container's FQDN and port.
    ```json
    {
     "db_host": "db",
     "db_port": 3306,
     "default_site": "frontend",
     "host_name": "http://frappe_docker-backend-1:8000",
     "redis_cache": "redis://redis-cache:6379",
     "redis_queue": "redis://redis-queue:6379",
     "redis_socketio": "redis://redis-queue:6379",
     "socketio_port": 9000
    }
    ```
    Save the file and exit the editor (`Ctrl+O`, `Enter`, `Ctrl+X` for nano), (ctrl+esc :wq for vi).

5.  **Verify Dependencies Inside the Container:**
    *   Check the `wkhtmltopdf` version. It should say "with patched Qt".
        ```bash
        wkhtmltopdf --version
        ```
    *   If it's missing or not the correct version, install it. Make sure to use the [patched Qt version](https://wkhtmltopdf.org/downloads.html).
        ```bash
        # Run if dependencies are missing
        apt-get install -y xvfb libfontconfig wkhtmltopdf #install wkhtmltopdf only if its not installed
        ```
    *   Check if `xvfb-run` is available:
        ```bash
        xvfb-run --help
        ```
6.  **Restart Docker Containers:**
    Back on your host machine, restart the containers for the changes to take effect.
    ```bash
    docker compose -f pwd.yml restart
    ```

#### Alternative Workaround
If the issue persists, switch the PDF generator to the more reliable Chrome engine.

1.  In ERPNext, go to **Print Format**.
2.  Find and duplicate the format you are using (e.g., "Sales Invoice").
3.  In the new duplicated format, change the **PDF Generator** from `Wkhtmltopdf` to **Chrome**.
4.  Save and use this new print format.

---

## Part 3: Future Work & Planning

*   [ ] **Connect ERPNext to Google Services:** Requires an HTTPS URL for OAuth callbacks.
*   [ ] **Obtain a Domain and SSL Certificate:** Plan to use a domain and set up SSL.
*   [ ] **Set up NGINX as a Reverse Proxy:** Use a dedicated NGINX container to handle public traffic and SSL termination for a production-ready setup.

> **Best Practice:** Avoid manually editing `site_config.json` for settings that can be configured via the UI or `bench` commands. Frappe often overwrites manual changes during updates.

---

## Appendix A: Bench CLI Cheatsheet

A quick reference for common `bench` commands, typically run inside the container (`docker exec -it <container_name> bash`).

### Bench Management
- **Start/Restart:** `bench start`, `bench restart`
- **Healthcheck:** `bench doctor`
- **Build JS Assets:** `bench build [--app my_app_name]`
- **Backup all sites:** `bench backup-all-sites`
- **Switch branches:** `bench switch-to-branch <branch_name> [app1 app2...]`

### Site Management
- **Create new site:** `bench new-site mysite.local`
- **Migrate database:** `bench --site mysite.local migrate`
- **Backup site:** `bench --site mysite.local backup`
- **Restore site:** `bench --site mysite.local restore <path-to-sql-file>`
- **Drop site:** `bench drop-site mysite.local [--no-backup --force]`

### App Management
- **Install app on site:** `bench --site mysite.local install-app my_app_name`
- **Uninstall app:** `bench --site mysite.local uninstall-app my_app_name`

### Miscellaneous
- **Show config:** `bench show-config`
- **List apps:** `bench --site mysite.local list-apps`
- **Update Bench CLI:** `bench update --bench`

---

## Appendix B: Manual NGINX, Domain, and SSL Setup (Non-Docker)

**Disclaimer:** The following steps are for a **traditional, non-Docker `bench` installation** on a bare server (e.g., a VPS). **Do not** follow these steps for the Docker setup described in Part 1. This is included for reference only.

### Setting up NGINX

1.  **Setup NGINX with Bench:** `bench setup nginx`
2.  **Restart Supervisor:** `sudo supervisorctl restart all`
3.  **Setup for Production:** `sudo bench setup production <your-linux-user>`
4.  **Configure Firewall (UFW):**
    ```bash
    sudo ufw allow 22,25,143,80,443,3306,8000/tcp
    sudo ufw enable
    ```

### Setting up Domain and SSL

1.  **Prerequisite:** Point an 'A' record for your domain to your server's IP address.
2.  **Navigate to Bench Directory:** `cd /home/<your-linux-user>/frappe-bench/`
3.  **Enable DNS Multitenancy:** `bench config dns_multitenant on`
4.  **Add Your Domain:** `bench setup add-domain YOUR_DOMAIN.com --site your_site_name.local`
5.  **Re-setup and Reload NGINX:** `bench setup nginx` and `sudo service nginx reload`
6.  **Install SSL with Certbot:**
    ```bash
    sudo snap install --classic certbot
    sudo ln -s /snap/bin/certbot /usr/bin/certbot
    sudo certbot --nginx
    ```
    Follow the on-screen prompts to complete the SSL installation.
