# CanteenGo: Campus Canteen Ordering System

SYSIN Final Project (System Architecture and Integration)
Group: (group name)

## 1. Description

CanteenGo shows the campus canteen menu to anyone, without an account. Only canteen staff log in to add and edit menu items. The system runs as Docker containers behind a single Nginx entry point, and Jenkins builds, tests and deploys it automatically on every push to `main`.

## 2. Architecture

(Insert the architecture diagram image here: `![Architecture](docs/architecture.png)`)

```
Browser --> proxy (Nginx :8080)
              |-- /            --> frontend   (Nginx, static pages)
              |-- /api/menu/   --> catalog-api (Spring Boot)   --+
              '-- /api/auth/   --> auth-api    (Express) --+--> db (MySQL)
                       [ network: canteen-net | volume: db-data ]

Developer -- git push --> GitHub -- webhook (ngrok) --> Jenkins
Jenkins: Checkout > Test > Build Images (tag = build #) > Deploy > Smoke Test
```

| Container | Technology | Role | Port |
| --- | --- | --- | --- |
| proxy | Nginx 1.27-alpine | Single entry point, routes by path | 80, published as 8080 |
| frontend | Static HTML/CSS/JS on Nginx | Public menu, staff login, staff admin page | 80 |
| catalog-api | Java 21, Spring Boot 3 | Menu item CRUD | 5000 |
| auth-api | Node.js 20, Express | Staff login, JWT, staff accounts | 3000 |
| db | MySQL 8.4 | Users and menu items, volume `db-data` | 3306 (not published) |
| jenkins | Jenkins LTS + Docker CLI | CI/CD server (separate compose) | 8080, published as 8081 |

## 3. Prerequisites

- Docker Desktop (Docker Engine with Compose v2)
- Git
- ngrok account and the ngrok client (for the GitHub webhook)
- A GitHub account with access to this repository

## 4. How to start Jenkins

```bash
cd infra/jenkins
docker compose up -d --build
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

1. Open http://localhost:8081 and paste the password.
2. Install the suggested plugins, then install **Docker Pipeline**.
3. Create the admin user.
4. Add a **Secret file** credential with ID `canteengo-env` (your real `.env`).
5. Create a Pipeline job: *Pipeline script from SCM*, this repository, branch `main`, Script Path `Jenkinsfile`.

## 5. How to start the stack manually

```bash
cp .env.example .env      # then edit the values
docker compose up -d --build
```

Open http://localhost:8080. Stop with `docker compose down` (data stays in the `db-data` volume).

Staff login uses the `ADMIN_USER` and `ADMIN_PASSWORD` values from `.env`.

## 6. How the trigger is configured

- **Primary:** GitHub webhook. Run `ngrok http 8081`, then in GitHub go to *Settings > Webhooks > Add webhook*. Payload URL is `https://<ngrok-url>/github-webhook/`, content type `application/json`, event: push. The Jenkinsfile uses `githubPush()`.
- **Fallback:** Poll SCM with `H/2 * * * *` in the job configuration.

## 7. Rollback

```bash
TAG=<previous build number> docker compose -p canteengo up -d --no-build
```

The footer of the site shows the build number that is live.

## 8. Team and roles

| Member | Role |
| --- | --- |
| (name) | Project Lead / Scrum Master |
| (name) | DevOps / CI-CD Engineer |
| (name) | Infrastructure Engineer |
| (name) | Backend and Database Engineer |
| (name) | Frontend, QA, and Documentation Lead |
