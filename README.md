# ERPNext-TailorMade

# ERPNext Docker with Ngrok and n8n: Local Host Installation and Custom Apps Setup Guide

This guide details the process of setting up a full ERPNext instance using Docker on a local host machine. It covers custom app installation, post-installation workarounds for errors, and setting up products, invoices, and custom apps specifically for a women's clothing store.

**Prerequisites:**
*   Docker and Docker Compose installed.

**Acknowledgement:**
This guide is heavily inspired by Atul Bhatt's YouTube video from the Know It Today channel: [# Frappe Docker Custom Apps Installation using Quick Build Image | 2025](https://www.youtube.com/watch?v=FWGWKC_rZeI&t=1134s).

**Note:** Most steps are mentioned in the official ERPNext documentation. Please refer to it first for any changes before following this guide. I will try to keep this guide up-to-date.
**Official Documentation Link:** [https://github.com/frappe/frappe_docker/tree/main](https://github.com/frappe/frappe_docker/tree/main)

---

## Step 1: Pull the Latest Version of ERPNext from GitHub

Clone the `frappe_docker` repository:
```bash
git clone https://github.com/frappe/frappe_docker
```

---

## Step 2: Create `apps.json` for Custom Applications

Navigate to the development directory:
```bash
cd frappe_docker/development
```

Remove the example `apps.json` if it exists:
```bash
rm -rf apps-example.json
```

Create a new `apps.json` file using your preferred text editor (e.g., `nano` or VS Code):
```bash
sudo nano apps.json
```
or
```bash
code apps.json
```

---

## Step 3: Load Custom Apps Through `apps.json` File

A Base64 encoded string of the `apps.json` file needs to be passed as a build argument environment variable.

Create the `apps.json` file with the Git URLs and branches for your desired apps.

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
    "url": "https://github.com/frappe/payments",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/ecommerce_integrations.git",
    "branch": "develop"
  },
  {
    "url": "https://github.com/frappe/hrms.git",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/insights.git",
    "branch": "main"
  },
  {
    "url": "https://github.com/frappe/crm.git",
    "branch": "main"
  },
  {
    "url": "https://github.com/frappe/print_designer.git",
    "branch": "main"
  },
  {
    "url": "https://github.com/resilient-tech/india-compliance",
    "branch": "version-15"
  }
]
```

**Notes:**
*   The `url` needs to be an HTTP(S) Git URL. For private repositories, use a Personal Access Token (PAT) without the username, e.g., `http://{{PAT}}@github.com/project/repository.git`.
*   Add dependencies manually in `apps.json`. For example, if you are installing `hrms`, ensure `erpnext` is also listed.
*   Use your fork or a specific branch for ERPNext if you need to use your fork or test a Pull Request (PR).

---

## Step 4: Generate Base64 String from `apps.json` and Validate

**Generate the Base64 string:**

Example:
```bash
export APPS_JSON_BASE64=$(base64 -w 0 /path/to/apps.json)
```

My production command (assuming `apps.json` is in the home directory):
```bash
export APPS_JSON_BASE64=$(base64 -w 0 ~/apps.json)
```

Or, if `apps.json` is in your current directory:
```bash
export APPS_JSON_BASE64=$(base64 -w 0 apps.json)
```

**Test the Output:**

To verify the previous step, decode the `APPS_JSON_BASE64` environment variable.

1.  Decode and save the output into a JSON file named `apps-test-output.json`:
    ```bash
    echo -n ${APPS_JSON_BASE64} | base64 -d > apps-test-output.json
    ```
2.  Open `apps-test-output.json` to review the JSON output and ensure the content is correct.
3.  Display the content using `cat`:
    ```bash
    cat apps-test-output.json
    ```

**Validate the `apps.json` file (optional, requires `jq`):**
```bash
jq empty apps.json
```

---

## Step 5: Build the Docker Image

Log in to Docker:
```bash
docker login
```

Navigate back to the `frappe_docker` directory if you are not already there:
```bash
cd ..
```

### Quick Build Image

This method uses pre-built `frappe/base:${FRAPPE_BRANCH}` and `frappe/build:${FRAPPE_BRANCH}` image layers, which include the required Python and NodeJS runtimes. This speeds up the build time. It uses `images/layered/Containerfile`.

**Method 1: using custom tag**

Example:
```bash
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=docker_username/frappe-custom:latest \
  --file=images/layered/Containerfile .
```

My production command:
```bash
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=riyann00b/frappe-custom:latest \
  --file=images/layered/Containerfile .
```

**Method 2: Manual Tagging (if Method 1 fails)**

Build with a temporary tag:
```bash
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=ghcr.io/user/repo/custom:1.0.0 \
  --file=images/layered/Containerfile .
```
*(Sit back, relax, and have a coffee – this will take a while.)*

Now, tag your build.
Example:
```bash
docker tag ghcr.io/user/repo/custom:1.0.0 docker_username/frappe-custom:latest
```

My production command:
```bash
docker tag ghcr.io/user/repo/custom:1.0.0 riyann00b/frappe-custom:latest
```

**Verify the image:**
```bash
docker images
```

If you haven't logged into Docker yet, do so now. You might see the old `ghcr.io/user/repo/custom:1.0.0` image still listed.

Remove it with:
```bash
docker rmi -f <tag_or_image_id_of_ghcr.io_image>
```

Now, ensure you are logged in:
```bash
docker login
```

**Push the image to use in YAML files:**
```bash
docker push riyann00b/frappe-custom:latest
```

---

## Step 6: Edit Your `pwd.yml`

If not in the `/frappe_docker` directory, navigate to it:
```bash
cd /frappe_docker # Or the correct path to your frappe_docker directory
```

Now, edit your `pwd.yml` file using your favorite text editor (e.g., `nano` or VS Code).
Example:
```bash
sudo nano pwd.yml
```
My production command:
```bash
code pwd.yml
```

In `pwd.yml`, replace all instances of the default image (e.g., `frappe/erpnext:v15.x.x`) with your custom image (e.g., `riyann00b/frappe-custom:latest`).

Around line 80 (this may vary), add your custom apps for installation in the `create-site` service command.
Example original line might look like:
```yaml
# ...
    command:
      - "bench"
      - "new-site"
      # ... other args ...
      - "--install-app"
      - "erpnext"
      # ... other args ...
# ...
```

Modify it to include your apps. The full command within the YAML might look something like this (adjust based on the actual structure of your `pwd.yml`):
```yaml
# Ensure this is part of the command arguments for the create-site service
# bench new-site --mariadb-user-host-login-scope='%' --admin-password=admin --db-root-username=root --db-root-password=admin --install-app erpnext --install-app your_custom_app --set-default frontend;

# My production apps to add (as individual --install-app flags):
# --install-app payments --install-app ecommerce_integrations --install-app print_designer --install-app india_compliance --install-app hrms --install-app insights --install-app crm
```
Find the line responsible for `bench new-site ...` and add `--install-app <app_name>` for each custom app.

For example, if the original command was:
`bench new-site ... --install-app erpnext ...`

Change it to (ensure proper YAML formatting for the command list):
`bench new-site ... --install-app erpnext --install-app payments --install-app ecommerce_integrations --install-app print_designer --install-app india_compliance --install-app hrms --install-app insights --install-app crm ...`

Save all the changes to `pwd.yml`.

---

## Step 7: Run Docker Compose and Install Apps

Run Docker Compose with your `pwd.yml` file:
```bash
docker compose -f pwd.yml up -d
```

Then, run this command to monitor the site creation and app installation logs:
```bash
docker logs frappe_docker-create-site-1 -f
```
*(The service name `frappe_docker-create-site-1` might vary based on your project name and setup. Check `docker ps` if unsure.)*

Once the process is complete, you should be able to access your ERPNext site.

**Access your ERPNext Site:**
Open your browser and go to: `http://localhost:8080`
