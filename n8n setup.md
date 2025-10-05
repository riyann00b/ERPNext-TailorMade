# ⚙️ n8n Setup and Automations

This guide covers setting up n8n for local use with PostgreSQL and explores different methods for deployment.

---

## 🛠️ Local-only n8n with PostgreSQL + Tunnel (Small Store Setup)

This setup is ideal for local development and testing, providing a robust PostgreSQL backend and a public tunnel for external access.

### 1. Install Docker & Docker Compose

Run these commands once to install Docker and Docker Compose:

```bash
sudo apt update && sudo apt install -y docker.io docker-compose
sudo systemctl enable docker --now
```

### Method 1: Running n8n with Docker Run (Quick Start)

This method is a quick way to get n8n running with a PostgreSQL database and a tunnel, useful for immediate testing.

```bash
docker volume create n8n_data

docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -e GENERIC_TIMEZONE="Asia/Kolkata" \
  -e TZ="Asia/Kolkata" \
  -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true \
  -e N8N_RUNNERS_ENABLED=true \
  -e DB_TYPE=postgresdb \
  -e DB_POSTGRESDB_DATABASE=<POSTGRES_DATABASE> \
  -e DB_POSTGRESDB_HOST=<POSTGRES_HOST> \
  -e DB_POSTGRESDB_PORT=<POSTGRES_PORT> \
  -e DB_POSTGRESDB_USER=<POSTGRES_USER> \
  -e DB_POSTGRESDB_SCHEMA=<POSTGRES_SCHEMA> \
  -e DB_POSTGRESDB_PASSWORD=<POSTGRES_PASSWORD> \
  -v n8n_data:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n \
  start --tunnel
```

**Note:** Replace the placeholder `<...>` values with your actual PostgreSQL connection details.

---

### Method 2: Using Docker Compose for a More Robust Setup

This method uses `docker-compose.yml` to manage the n8n service, PostgreSQL, Redis, and a worker for better scalability and management.

#### 2. Create a Project Folder

```bash
mkdir ~/n8n && cd ~/n8n
```

#### 3. Create `docker-compose.yml`

Paste the following content into a file named `docker-compose.yml` in your `~/n8n` directory:

```yaml
version: '3.8'

volumes:
  db_storage:
  n8n_storage:
  redis_storage:

x-shared: &shared
  restart: always
  image: docker.n8n.io/n8nio/n8n
  environment:
    - DB_TYPE=postgresdb
    - DB_POSTGRESDB_HOST=postgres
    - DB_POSTGRESDB_PORT=5432
    - DB_POSTGRESDB_DATABASE=${POSTGRES_DB}
    - DB_POSTGRESDB_USER=${POSTGRES_NON_ROOT_USER}
    - DB_POSTGRESDB_PASSWORD=${POSTGRES_NON_ROOT_PASSWORD}
    - EXECUTIONS_MODE=queue
    - QUEUE_BULL_REDIS_HOST=redis
    - QUEUE_HEALTH_CHECK_ACTIVE=true
    - N8N_ENCRYPTION_KEY=${ENCRYPTION_KEY}
  links:
    - postgres
    - redis
  volumes:
    - n8n_storage:/home/node/.n8n
  depends_on:
    redis:
      condition: service_healthy
    postgres:
      condition: service_healthy

services:
  postgres:
    image: postgres:16
    restart: always
    environment:
      - POSTGRES_USER
      - POSTGRES_PASSWORD
      - POSTGRES_DB
      - POSTGRES_NON_ROOT_USER
      - POSTGRES_NON_ROOT_PASSWORD
    volumes:
      - db_storage:/var/lib/postgresql/data
      - ./init-data.sh:/docker-entrypoint-initdb.d/init-data.sh
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -h localhost -U ${POSTGRES_USER} -d ${POSTGRES_DB}']
      interval: 5s
      timeout: 5s
      retries: 10

  redis:
    image: redis:6-alpine
    restart: always
    volumes:
      - redis_storage:/data
    healthcheck:
      test: ['CMD', 'redis-cli', 'ping']
      interval: 5s
      timeout: 5s
      retries: 10

  n8n:
    <<: *shared
    command: start --tunnel
    ports:
      - 5678:5678

  n8n-worker:
    <<: *shared
    command: worker
    depends_on:
      - n8n
```

#### Environment Variables

Create a `.env` file in the same directory as your `docker-compose.yml` file and add the following variables. **Remember to change `changeEncryptionKey` to a strong, unique key.**

```dotenv
POSTGRES_USER=n8n_user
POSTGRES_PASSWORD=1234
POSTGRES_DB=n8n

POSTGRES_NON_ROOT_USER=n8n_user1
POSTGRES_NON_ROOT_PASSWORD=1234

ENCRYPTION_KEY=changeEncryptionKey
```

#### 4. Start n8n

Start the n8n services in detached mode:

```bash
docker compose up -d
```

To check the logs and find your tunnel URL:

```bash
docker logs -f n8n
```

You should see output similar to this, indicating your public tunnel URL:

```
Tunnel URL: https://random-subdomain.n8n.cloud/
```

---

#### 📂 n8n Folder Structure

You can find the complete configuration for this setup in my n8n folder: [Link](https://github.com/riyann00b/ERPNext-TailorMade/tree/main/n8n).

