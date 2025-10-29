# DevOps Report - Django PostgreSQL Deployment

**Project:** Django Application with PostgreSQL Database  

---

## 1. Technologies Used

### Backend & Framework
- **Django** — Python web framework  
- **Python 3.11** — Programming language  
- **PostgreSQL 15** — Relational database  

### Containerization
- **Docker** — Container platform  
- **Docker Compose** — Multi-container orchestration  

### CI/CD
- **GitHub Actions** — Continuous Integration / Continuous Deployment  
- **Railway** — Cloud deployment platform  
- **Docker Hub** — Container registry  

### Testing & Security
- **Flake8** — Python code linter  
- **Bandit** — Security vulnerability scanner  
- **Django Test Framework** — Unit testing  

---

## 2. CI/CD Pipeline Design

### Summary
A 5-stage GitHub Actions pipeline for a Dockerized Django app:

1. **Build & Install**
2. **Lint & Security Scan**
3. **Test** (with PostgreSQL service)
4. **Build Docker Image**
5. **Deploy** (Railway) — runs only on `main` after successful tests

**Goals:**  
- CI/CD automation  
- Secure secrets handling  
- Automatic migrations on deploy  
- Visible logs  


### Pipeline Stages

1. **Build & Install**
   - Checkout code  
   - Set up Python  
   - Install dependencies from `requirements.txt`  

2. **Lint & Security Scan**
   - Run `flake8` (linting)
   - Run `bandit` (security scan)

3. **Test (with PostgreSQL service)**
   - Start PostgreSQL container service  
   - Wait for database health (`pg_isready`)  
   - Run migrations and Django tests  

4. **Build Docker Image**
   - Build Docker image  
   - Authenticate and push image to Docker Hub (optional if using Railway directly)

5. **Deploy (Railway)**
   - Use Railway CLI or GitHub Action  
   - Use `RAILWAY_TOKEN` and `RAILWAY_SERVICE_NAME` secrets  
   - Run deployment (`railway up --service <service-name>`)  
   - Stream deployment logs 

### Pipeline Flow
1. **Trigger:** Push to `main` or `develop` branch
2. **Sequential Execution:** Each stage depends on previous stage success
3. **Conditional Deploy:** Only runs on `main` branch
4. **Failure Handling:** Pipeline stops if any stage fails


## 3. Secret Management Strategy

### GitHub Secrets

|      Secret Name      | Purpose                    | Usage |
|-----------------------|----------------------------|-------------------------|
| `DOCKERHUB_USERNAME`  | Docker Hub authentication  | Push images to registry |
| `DOCKERHUB_TOKEN`     | Docker Hub access token    | Secure authentication |
| `RAILWAY_TOKEN`       | Railway API authentication | Deploy to Railway |
| `RAILWAY_PROJECT_ID`  | Railway project identifier | Link correct project |
| `DB_NAME`             | Database name              | Test environment |
| `DB_USER`             | Database user              | Test environment |
| `DB_PASSWORD`         | Database password          | Test environment |


### Environment Variables

**Local Development (.env file):**
```bash
DB_NAME=carsdb
DB_USER=django_user
DB_PASSWORD=django_password
DB_HOST=db
DB_PORT=5432
```

**Railway Production:**
```bash
DB_NAME=${{Postgres.PGDATABASE}}
DB_USER=${{Postgres.PGUSER}}
DB_PASSWORD=${{Postgres.PGPASSWORD}}
DB_HOST=${{Postgres.PGHOST}}
DB_PORT=${{Postgres.PGPORT}}
```

### Best Practices
- ✅ Never commit secrets to Git  
- ✅ Use GitHub Secrets for CI/CD  
- ✅ Use `.env` (gitignored) for local development  
- ✅ Use Railway-managed environment variables for production  

---

## 4. Testing Process

### Local Testing

**Unit Tests:**
```bash
python manage.py test
```

**Database Connection Test:**
```bash
python manage.py migrate
python manage.py dbshell
```

### CI/CD Testing

- PostgreSQL 15 (Alpine) container runs as a GitHub Actions service  
- Health checks ensure DB is ready before running tests  
- Django connects using environment variables  
- Migrations run automatically before executing tests  

**Workflow Sequence:**
1. PostgreSQL service starts with health checks  
2. Django connects to test DB  
3. Run migrations  
4. Execute unit tests  
5. Destroy service container after completion  

---

## 5. Key Commands

**Local / Docker**
```bash
docker build -t myapp:latest .
docker-compose up -d
docker-compose push app
```

**Django**
```bash
python manage.py migrate
python manage.py test
python manage.py loaddata data.json
```

**Railway**
```bash
npm i -g @railway/cli
railway login
railway up --service <service-name>
railway logs
```

**GitHub**
```bash
git push origin main
```
---

## 6. Lessons Learned

### 1. Docker Compose vs Railway Deployment
- **Issue:** Confusion between local `docker-compose.yml` and production setup.  
- **Solution:** Use Docker Compose for local only; Railway uses the `Dockerfile` and managed Postgres.  
- **Lesson:** Separate dev and prod infrastructure cleanly.

### 2. PostgreSQL Image Push Permissions
- **Issue:** Couldn’t push `postgres:15-alpine` to Docker Hub.  
- **Solution:** Only push your app’s image. Use managed DBs (Railway, RDS, etc.) in prod.  
- **Lesson:** Don’t manage databases manually in production.

### 3. Database Host Configuration
- **Issue:** Django connecting to `localhost` inside Docker.  
- **Solution:** Use service name (`db`) in `.env`.  
- **Lesson:** Containers communicate by service names, not `localhost`.

### 4. Flake8 Linting Failures
- **Issue:** Migration files triggered false lint errors.  
- **Solution:** Exclude `migrations/` in `.flake8`.  
- **Lesson:** Configure linters to respect framework-generated files.

### 5. Data Migration Between Environments
- **Issue:** Local DB data not transferred to Railway.  
- **Solution:** Use Django fixtures or seed scripts.  
- **Lesson:** Schema auto-migrates; data must be handled explicitly.


