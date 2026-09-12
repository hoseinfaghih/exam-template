# Scenario 1

## Problem 1: Docker Compose was not installed

**How I found it:**

I checked whether Docker Compose was available by running `docker compose version`. The command was not recognized, so I searched the available packages and found `docker-compose-v2`.

**What was wrong:**

Docker was installed, but the `docker compose` command was not available.

**How I fixed it:**

I installed the `docker-compose-v2` package.

**Commands I used:**

```bash
docker compose version
apt search docker-compose
sudo apt install docker-compose-v2
docker compose version
```

## Problem 2: Backend was not connected to PostgreSQL

**How I found it:**

After starting the containers, the backend logs showed that it could not resolve the hostname `db`. I checked the Docker networks and found that the backend was only connected to `nginx-backend-net`, while PostgreSQL was connected to `backend-db-net`.

**What was wrong:**

The backend container was not connected to the same Docker network as PostgreSQL.

**How I fixed it:**

I added `backend-db-net` to the backend service in `docker-compose.yml`.

**Config I changed (only the changed part):**

```yaml
backend:
  networks:
    - nginx-backend-net
    - backend-db-net
```

## Problem 3: Nginx was using the wrong backend address

**How I found it:**

I tested `curl http://localhost/graph` and received `502 Bad Gateway`. I then checked the Nginx logs, which showed that `backend-api` could not be resolved. I also checked the Compose configuration and found that the actual backend service was named `backend` and exposed port `5000`.

**What was wrong:**

Nginx was configured to connect to `backend-api:8080`, but the backend service was named `backend` and was listening on port `5000`.

**How I fixed it:**

I changed the Nginx upstream address from `backend-api:8080` to `backend:5000`.

**Config I changed (only the changed part):**

```nginx
set $backend_upstream http://backend:5000;
```

## Final step

I rebuilt and restarted the services:

```bash
docker compose down
docker compose up -d --build
docker compose ps
curl -i http://localhost/graph
```

The final request returned **HTTP 200 OK**.
