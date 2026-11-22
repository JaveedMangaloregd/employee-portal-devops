# Employee Portal --- AWS Cloud Deployment & CI/CD

A Flask-based Employee Management Portal deployed on **AWS EC2** using
**Docker** and an automated **GitLab CI/CD pipeline**.

This project demonstrates containerization, automated testing, Docker
image management, continuous integration and deployment, staging and
production environments, and persistent application storage.

## Table of Contents

-   [Project Overview](#project-overview)
-   [Application Features](#application-features)
-   [Technology Stack](#technology-stack)
-   [Architecture](#architecture)
-   [Infrastructure](#infrastructure)
-   [CI/CD Pipeline](#cicd-pipeline)
-   [Deployment Environments](#deployment-environments)
-   [Database Persistence](#database-persistence)
-   [Project Structure](#project-structure)
-   [Local Setup](#local-setup)
-   [Useful EC2 Commands](#useful-ec2-commands)
-   [GitLab CI/CD Configuration](#gitlab-cicd-configuration)
-   [Security Considerations](#security-considerations)
-   [Troubleshooting](#troubleshooting)
-   [Future Improvements](#future-improvements)

## Project Overview

The Employee Portal allows users to manage employee records through a
web interface. The DevOps objective is to automate the application
lifecycle: source control, code quality checks, unit testing, Docker
image creation, image publishing, and deployment to AWS EC2.

The workflow includes: 1. Source code management using Git. 2. Automated
code quality checks using Flake8. 3. Automated unit testing using
Pytest. 4. Docker image creation. 5. Publishing images to GitLab
Container Registry. 6. Automated staging and production deployments to
AWS EC2. 7. Persistent SQLite storage using Docker named volumes.

## Application Features

-   View employee records.
-   Add new employees.
-   Edit existing employee details.
-   Delete employee records.
-   Store employee information using SQLite and Flask-SQLAlchemy.
-   Render a web interface using Flask templates.

## Technology Stack

  Category               Technology
  ---------------------- ---------------------------
  Programming Language   Python
  Web Framework          Flask
  Database               SQLite
  ORM                    Flask-SQLAlchemy
  Application Server     Gunicorn
  Containerization       Docker
  Container Registry     GitLab Container Registry
  CI/CD                  GitLab CI/CD
  Cloud Platform         AWS
  Compute                Amazon EC2
  Operating System       Ubuntu Server 24.04 LTS
  CI/CD Runner           GitLab Runner
  Testing                Pytest
  Code Quality           Flake8
  Version Control        Git

## Architecture

``` text
                  DEVELOPER
                      |
               Local Development
                  (WSL / VS Code)
                      |
                   Git Push
                      |
                      v
              GITLAB REPOSITORY
                      |
                      v
               GITLAB CI/CD
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
       Flake8       Pytest      Docker Build
          |           |              |
          +-----------+--------------+
                      |
                      v
             GITLAB CONTAINER
                 REGISTRY
                      |
                      v
                  AWS EC2
                      |
               GitLab Runner
                      |
             +--------+--------+
             |                 |
             v                 v
          STAGING           PRODUCTION
          Port 8081          Port 80
             |                 |
             +--------+--------+
                      |
                Flask + Gunicorn
                      |
                   SQLite DB
                      |
                 Docker Volumes
```

### Deployment Flow

1.  A developer pushes changes to GitLab.
2.  GitLab CI/CD starts the pipeline.
3.  Flake8 checks Python code.
4.  Pytest executes unit tests.
5.  Docker builds the application image.
6.  The image is pushed to GitLab Container Registry.
7.  GitLab Runner executes deployment jobs on AWS EC2.
8.  Staging is deployed and tested.
9.  Changes merged into `main` trigger production deployment.
10. The production container uses persistent database storage.

## Infrastructure

### AWS EC2

  Configuration       Value
  ------------------- -------------------------
  Cloud Provider      AWS
  Region              Asia Pacific (Mumbai)
  Region Code         `ap-south-1`
  Operating System    Ubuntu Server 24.04 LTS
  Instance Type       `t3.micro`
  EBS Storage         8 GB
  Container Runtime   Docker
  CI/CD Runner        GitLab Runner

### Network access

  Protocol       Port Intended access
  ------------ ------ ----------------------------------
  SSH              22 Restricted to authorized access
  HTTP             80 Public application access
  HTTPS           443 Reserved for future HTTPS setup
  Custom TCP     8081 Staging; restrict to trusted IPs

HTTPS and SSL configuration are future improvements, not yet
implemented.

## CI/CD Pipeline

GitLab CI/CD automates testing, building, and deployment.

  Pipeline stage              Purpose
  --------------------------- --------------------------------------
  Test                        Run code quality and unit tests
  Build                       Build and publish the Docker image
  Deploy Staging              Deploy the application to staging
  Deploy Feature              Deploy feature branches for testing
  Automated Feature Testing   Validate feature deployments
  Automated Testing           Validate staging deployment
  Deploy Production           Deploy the application to production

### Code quality --- Flake8

Flake8 checks: - `app.py` - `models/models.py` - `test_app.py`

### Automated testing --- Pytest

The test suite covers: - Homepage response. - Homepage content. -
Employee creation. - Employee editing. - Employee deletion.

### Docker image and registry

The application is packaged into a Docker image tagged with the Git
commit short SHA:

``` text
employee-image:<commit-short-sha>
```

The image is pushed to GitLab Container Registry. AWS EC2 pulls the
required image during deployment.

### Automated deployment

The deployment jobs: - Connect to EC2 using SSH. - Authenticate with
GitLab Container Registry. - Pull the selected image. - Replace the
existing application container. - Start the updated container. -
Initialize database tables. - Mount persistent database storage.

## Deployment Environments

  Environment   Container name                   Host port   Container port
  ------------- ------------------------------ ----------- ----------------
  Staging       `employee-portal-staging`             8081             5000
  Production    `employee-portal-production`            80             5000

Staging is used to validate changes. The current pipeline deploys
staging from `main`; production deployment is also triggered for changes
merged into `main`.

## Database Persistence

The application uses SQLite with Flask-SQLAlchemy. The database path
inside the container is:

``` text
/app/instance/emp_db.db
```

Docker named volumes keep the database outside the replaceable container
filesystem.

**Staging volume**

``` text
employee-portal-db
```

**Production volume**

``` text
employee-portal-production-db
```

Volume mounts:

``` bash
-v employee-portal-db:/app/instance
```

``` bash
-v employee-portal-production-db:/app/instance
```

Database tables are initialized during deployment. The staging
initialization command is:

``` bash
docker exec employee-portal-staging python -c \
"from app import app, db; ctx=app.app_context(); ctx.push(); db.create_all()"
```

Production performs the equivalent initialization for its container.

**Important:** Docker volumes provide persistence across container
replacements, but they are not independent backups. A separate backup
process is still required.

## Project Structure

``` text
employee-portal/
├── app.py
├── models/
│   └── models.py
├── templates/
│   ├── index.html
│   ├── add.html
│   └── edit.html
├── static/
│   └── template.css
├── test_app.py
├── Dockerfile
├── Procfile
├── requirements.txt
├── .flake8
├── .gitignore
└── .gitlab-ci.yml
```

## Local Setup

### Prerequisites

-   Python 3.8 or a compatible environment.
-   Git.
-   Docker (for container workflows).
-   GitLab account.

### 1. Clone the repository

``` bash
git clone https://gitlab.com/self-group2060947/employee-portal
cd employee-portal
```

### 2. Create and activate a virtual environment

``` bash
python3 -m venv venv
source venv/bin/activate
```

On Windows PowerShell:

``` powershell
venv\Scripts\activate
```

### 3. Install dependencies

``` bash
pip install -r requirements.txt
```

### 4. Run the application

``` bash
python app.py
```

Open `http://localhost:5000/`.

### 5. Run tests and code quality checks

``` bash
pip install pytest flake8
pytest -v
flake8 app.py models/models.py test_app.py
```

## Useful EC2 Commands

Check running containers:

``` bash
docker ps
```

List all containers:

``` bash
docker ps -a
```

View application logs:

``` bash
docker logs --tail 100 employee-portal-production
docker logs --tail 100 employee-portal-staging
```

Restart production:

``` bash
docker restart employee-portal-production
```

List Docker volumes:

``` bash
docker volume ls
```

Inspect the production volume:

``` bash
docker volume inspect employee-portal-production-db
```

Test production locally from the EC2 instance:

``` bash
curl -I http://localhost/
```

Test staging locally from the EC2 instance:

``` bash
curl -I http://localhost:8081/
```

## GitLab CI/CD Configuration

The project uses the following GitLab CI/CD variables:

  Variable                 Purpose
  ------------------------ -------------------------------------
  `EC2_HOST`               EC2 private IP address
  `EC2_USER`               EC2 SSH username
  `EC2_SSH_PRIVATE_KEY`    SSH private key used for deployment
  `CI_REGISTRY`            GitLab Container Registry address
  `CI_REGISTRY_USER`       Registry authentication username
  `CI_REGISTRY_PASSWORD`   Registry authentication password

Sensitive values must be stored in GitLab CI/CD settings, never
committed to the repository or printed in job logs.

## Security Considerations

-   Restrict SSH access using security group rules.
-   Store deployment credentials in protected GitLab CI/CD variables.
-   Keep the deployment private key out of the repository.
-   Restrict access to the staging port.
-   Keep database files in Docker volumes.
-   Maintain separate staging and production containers.

HTTPS, independent database backups, and stronger production secret
management remain planned improvements.

## Troubleshooting

### Container is not running

``` bash
docker ps -a
docker logs employee-portal-production
```

### Application returns HTTP 500

``` bash
docker logs --tail 100 employee-portal-production
```

### Check database tables

``` bash
docker exec employee-portal-production python -c \
"from app import app, db; ctx=app.app_context(); ctx.push(); print(db.inspect(db.engine).get_table_names())"
```

### Initialize missing database tables

``` bash
docker exec employee-portal-production python -c \
"from app import app, db; ctx=app.app_context(); ctx.push(); db.create_all()"
```

### GitLab deployment failure

Check: 1. GitLab Runner status and tags. 2. Branch protection and CI/CD
variable availability. 3. SSH connectivity. 4. Container Registry
authentication. 5. Docker container logs.

## Future Improvements

-   [ ] Configure a custom domain.
-   [ ] Implement HTTPS using an SSL certificate.
-   [ ] Automate SQLite database backups.
-   [ ] Configure Prometheus monitoring.
-   [ ] Integrate Grafana dashboards.
-   [ ] Add EC2 resource monitoring.
-   [ ] Configure centralized application logging.
-   [ ] Improve deployment rollback strategy.
-   [ ] Explore Terraform for infrastructure provisioning.
-   [ ] Evaluate a managed database for larger-scale workloads.

## Author

**Employee Portal --- DevOps Project**

A hands-on cloud and DevOps engineering project demonstrating:

**AWS \| EC2 \| Linux \| Docker \| GitLab CI/CD \| GitLab Runner \|
Container Registry \| Python \| Flask \| Pytest \| Flake8 \| SQLite**

------------------------------------------------------------------------

*This project is evolving as additional DevOps tools and cloud
infrastructure capabilities are implemented.*
