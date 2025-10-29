# Django + PostgreSQL (Docker) — Project README


---

## What this repo contains
- A Django app (cars) using PostgreSQL
- Dockerfile and docker-compose.yml for local development
- GitHub Actions workflow: build → lint/security → test (Postgres) → build image → deploy (Railway)
- Helpers for automated migrations and optional data seeding

---

## Quick prerequisites
- Docker & Docker Compose (for containerized local run)
- Python 3.11 (if running locally without Docker)
- GitHub repo with Actions enabled
- Railway account (for production deployment)

---

## Local setup (minimal)

1. Create local DB user/database (only if running Django against local Postgres):
```sql
CREATE DATABASE carsdb;
CREATE USER carsadmin WITH ENCRYPTED PASSWORD 'carspass';
GRANT ALL PRIVILEGES ON DATABASE carsdb TO carsadmin;
```

2. Clone and enter repo:
```bash
git clone <repo-url>
cd <repo>
```

3. Create .env (gitignored) — example:
```env
DB_NAME=carsdb
DB_USER=carsadmin
DB_PASSWORD=carspass
DB_HOST=db
DB_PORT=5432
PORT=5000
```

4A. Run with Docker Compose (recommended):
```bash
docker compose up --build
# then (optional) create superuser / run migrations:
docker compose exec app python manage.py migrate
docker compose exec app python manage.py createsuperuser
```

4B. Or run locally with venv:
```bash
python -m venv venv
# Windows:
venv\Scripts\activate
# Unix:
source venv/bin/activate
pip install -r requirements.txt
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver 0.0.0.0:5000
```

Open: http://localhost:5000 (admin: /admin)

---

## Migrations, seed & sample data
- Make and apply migrations:
```bash
python manage.py makemigrations
python manage.py migrate
```
- To export local data:
```bash
python manage.py dumpdata > data.json
```
- To import into Railway or another env:
```bash
python manage.py loaddata data.json
```
- You can also add a small management `seed` command and run it during container start to auto-populate rows.

---

## Testing & linting
- Run tests:
```bash
python manage.py test
# or in Docker:
docker compose exec app python manage.py test
```
- Lint/security (recommended):
```bash
pip install flake8 bandit
flake8 .   # adapt .flake8 to ignore migrations if needed
bandit -r .
```

---

## CI/CD summary (GitHub Actions)
Pipeline stages:
1. Build & Install — setup Python, install deps  
2. Lint & Security — flake8, bandit (configurable non-blocking)  
3. Test — starts a Postgres service in Actions, runs migrations and tests  
4. Build Docker image — builds app image and optionally pushes to Docker Hub  
5. Deploy — runs only on `main`; deploys to Railway and streams logs

Keep secrets in GitHub Settings → Secrets and variables → Actions.

---

## Railway deployment (concise setup)
1. Create Railway account and new project → Add PostgreSQL plugin (Railway provisions DB).  
2. Add a service (connect GitHub repo or deploy via image). Railway shows service name (top of service card).  
3. Ensure your Dockerfile runs migrations on start, or run migrations from the deploy step:
```dockerfile
CMD python manage.py migrate && python manage.py runserver 0.0.0.0:$PORT
```
4. Create Railway token: Account → Tokens → Create token.  
5. Add GitHub secrets:
- `RAILWAY_TOKEN` — token
- `RAILWAY_SERVICE_NAME` — service name
- `DOCKERHUB_USERNAME` / `DOCKERHUB_TOKEN` — if pushing images
6. Push to `main` to trigger deploy via Actions.

---

## Common commands (cheat sheet)
- Build & run: docker compose up --build  
- Stop & remove: docker compose down -v  
- Build image: docker build -t <user>/django-app:latest .  
- Push image: docker login && docker push <user>/django-app:latest  
- Migrate: python manage.py migrate  
- Tests: python manage.py test  
- Dump/load data: dumpdata / loaddata  
- Railway CLI: npm i -g @railway/cli; railway up --service <service>; railway logs

---

## Project structure 
```
.
├── .github/workflows/ci-cd.yml
├── Dockerfile
├── docker-compose.yml
├── manage.py
├── myproject/
├── cars/
├── requirements.txt
├── .env (gitignored)
├── .flake8
└── README.md
```