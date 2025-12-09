# DevOps Capstone Project - Accounts Microservice

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Python 3.9](https://img.shields.io/badge/Python-3.9-green.svg)](https://shields.io/)

This repository contains a complete DevOps implementation for an Accounts microservice, developed as part of the [**IBM-CD0285EN-SkillsNetwork DevOps Capstone Project**](https://www.coursera.org/learn/devops-capstone-project?specialization=devops-and-software-engineering) in the [**IBM DevOps and Software Engineering Professional Certificate**](https://www.coursera.org/professional-certificates/devops-and-software-engineering).

## Project Overview

This project implements a full DevOps pipeline for a RESTful API microservice that manages user accounts. The implementation includes:

- **Full CRUD Operations**: Create, Read, Update, Delete, and List accounts
- **Test-Driven Development**: 95%+ code coverage with unit and integration tests
- **Security Features**: Flask-Talisman for HTTPS enforcement, CORS support
- **Containerization**: Docker image with non-root user
- **Kubernetes Deployment**: Multi-replica deployment with PostgreSQL backend
- **CI/CD Pipeline**: Automated testing, linting, building, and deployment

## Technology Stack

- **Backend**: Python 3.9, Flask 2.1.2
- **Database**: PostgreSQL (via psycopg2-binary)
- **ORM**: SQLAlchemy 1.4.46
- **Testing**: nose, coverage, factory-boy
- **Linting**: flake8, pylint, black
- **Server**: Gunicorn 20.1.0
- **Containerization**: Docker, Buildah
- **Orchestration**: Kubernetes/OpenShift
- **CI/CD**: GitHub Actions, Tekton Pipelines

## Architecture

### Data Model

The Account model contains the following fields:

| Name | Type | Optional |
|------|------|----------|
| id | Integer| False |
| name | String(64) | False |
| email | String(64) | False |
| address | String(256) | False |
| phone_number | String(32) | True |
| date_joined | Date | False |

### REST API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/` | Health check endpoint |
| POST | `/accounts` | Create a new account |
| GET | `/accounts/<id>` | Read an account by ID |
| PUT | `/accounts/<id>` | Update an account by ID |
| DELETE | `/accounts/<id>` | Delete an account by ID |
| GET | `/accounts` | List all accounts |

### Project Structure

```text
├── service/                <- microservice package
│   ├── common/            <- common log and error handlers
│   ├── config.py          <- Flask configuration object
│   ├── models.py          <- code for the persistent model
│   └── routes.py          <- code for the REST API routes
├── tests/                 <- folder for all tests
│   ├── factories.py       <- test factories
│   ├── test_cli_commands.py
│   ├── test_models.py
│   └── test_routes.py
├── deploy/                <- Kubernetes manifests
│   ├── deployment.yaml    <- Kubernetes Deployment (3 replicas)
│   └── service.yaml       <- Kubernetes Service (NodePort)
├── tekton/                <- Tekton pipeline definitions
│   ├── pipeline.yaml      <- CD pipeline definition
│   ├── tasks.yaml         <- Custom Tekton tasks
│   └── pvc.yaml          <- PersistentVolumeClaim
├── Dockerfile             <- Container image definition
└── .github/workflows/     <- GitHub Actions CI pipeline
    └── ci-build.yaml      <- CI workflow
```

## Development Workflow

### Sprint 1: Database & CRUD Operations
- Implemented database connection with PostgreSQL
- Created Account model with SQLAlchemy
- Developed REST API endpoints for all CRUD operations
- Achieved 95% test coverage with TDD approach

### Sprint 2: Security & CI Pipeline
- Added Flask-Talisman for security headers
- Implemented CORS support
- Created GitHub Actions CI pipeline with:
  - Linting (flake8)
  - Unit tests (nose with 95% coverage requirement)
  - Code quality checks

### Sprint 3: Containerization & Kubernetes
- Created production Dockerfile with:
  - Python 3.9-slim base image
  - Non-root user (UID 1000)
  - Gunicorn WSGI server
- Built and pushed Docker image to IBM Cloud Container Registry
- Deployed to OpenShift/Kubernetes with:
  - 3-replica deployment for high availability
  - PostgreSQL database with secret management
  - NodePort service for external access

### Sprint 4: CD Pipeline with Tekton
- Implemented automated CD pipeline with 6 stages:
  1. **init**: Workspace cleanup
  2. **clone**: Git repository clone
  3. **lint**: Flake8 code linting (runs in parallel with tests)
  4. **tests**: Unit tests with nose (runs in parallel with lint)
  5. **build**: Docker image build with Buildah
  6. **deploy**: Automated deployment to OpenShift
- Created custom Tekton task for running nosetests
- Configured pipeline parameters for flexible deployments

## Setup & Installation

### Development Environment

These labs are designed to be executed in the IBM Developer Skills Network Cloud IDE with OpenShift.

Initialize the environment:

```bash
source bin/setup.sh
```

This will:
- Install Python 3.9
- Create and activate a virtual environment
- Install all dependencies

Your prompt should look like:
```bash
(venv) theia:project$
```

### Installing Dependencies

```bash
make install
```

### Starting PostgreSQL

```bash
make db
```

### Running Tests

```bash
# Run all tests
nosetests

# Run with coverage
nosetests --with-coverage --cover-package=service
```

### Running the Service Locally

```bash
# Using Flask development server
flask run

# Using Gunicorn (production)
gunicorn --bind=0.0.0.0:8080 --log-level=info service:app
```

## Docker & Kubernetes

### Building the Docker Image

```bash
docker build -t accounts:1 .
```

### Running with Docker

```bash
docker run -p 8080:8080 \
  -e DATABASE_URI=postgresql://user:pass@host/db \
  accounts:1
```

### Deploying to Kubernetes

```bash
# Apply manifests
kubectl apply -f deploy/

# Check deployment status
kubectl get pods -l app=accounts
kubectl get svc accounts
```

## Tekton CI/CD Pipeline

### Pipeline Parameters

- `repo-url`: GitHub repository URL
- `branch`: Git branch to deploy (default: main)
- `build-image`: Full image name with registry path

### Running the Pipeline

```bash
tkn pipeline start cd-pipeline \
  -p repo-url="https://github.com/MironCo/devops-capstone-project.git" \
  -p branch=cd-pipeline \
  -p build-image=image-registry.openshift-image-registry.svc:5000/$SN_ICR_NAMESPACE/accounts:1 \
  -w name=pipeline-workspace,claimName=pipelinerun-pvc \
  -s pipeline \
  --showlog
```

### Pipeline Tasks

1. **cleanup** - Removes old workspace files
2. **git-clone** (Tekton Hub) - Clones repository
3. **flake8** (Tekton Hub) - Lints code with flake8
4. **nose** (custom) - Runs unit tests with coverage
5. **buildah** (ClusterTask) - Builds and pushes Docker image
6. **openshift-client** (ClusterTask) - Deploys to OpenShift

### Custom Tekton Task: nose

Located in `tekton/tasks.yaml`, this task:
- Installs Python dependencies
- Runs nosetests with configurable arguments
- Supports custom database URI for testing
- Reports code coverage

## GitHub Actions CI Pipeline

Located in `.github/workflows/ci-build.yaml`

**Triggers**: Push and Pull Request events

**Jobs**:
1. **Lint**: Runs flake8 with complexity and line-length checks
2. **Test**: Runs nosetests with 95% coverage requirement

Both jobs use PostgreSQL service container for integration tests.

## Security Features

- **Flask-Talisman**: Enforces HTTPS, sets security headers
- **CORS**: Cross-Origin Resource Sharing support
- **Non-root Container**: Docker container runs as user `theia` (UID 1000)
- **Secret Management**: Kubernetes secrets for database credentials
- **Environment Variables**: Sensitive data via environment configuration

## Testing Strategy

- **Unit Tests**: Model and route testing with 95% coverage
- **Integration Tests**: Full API testing with test database
- **Test Factories**: factory-boy for generating test data
- **Coverage Reports**: Automated coverage reporting in CI/CD
- **Spec-style Output**: Clear test descriptions with pinocchio

## Results

- **Final Grade**: 94%
- **Test Coverage**: 93% (253 statements, 17 missed)
- **CI Pipeline**: All checks passing
- **CD Pipeline**: Fully automated deployment
- **Production Deployment**: 3 replicas running on OpenShift

## Key Achievements

- Implemented complete CRUD API with Flask and SQLAlchemy
- Achieved 95%+ test coverage using TDD methodology
- Built secure, containerized microservice
- Created automated CI/CD pipeline with GitHub Actions and Tekton
- Deployed multi-replica application to Kubernetes/OpenShift
- Implemented parallel pipeline execution for faster builds
- Managed secrets and configuration properly in production

## Local Kubernetes Development

For local development with K3D and Tekton:

1. Bring up local K3D Kubernetes cluster:
   ```bash
   make cluster
   ```

2. Install Tekton:
   ```bash
   make tekton
   ```

3. Install ClusterTasks:
   ```bash
   make clustertasks
   ```

## Contributing

This project follows the IBM DevOps Capstone project structure. All development was done using:
- Test-Driven Development (TDD)
- Agile methodology with ZenHub kanban board
- Git workflow with feature branches and pull requests
- Continuous Integration and Continuous Deployment

## Author

Project completed by Miron as part of the IBM DevOps and Software Engineering Professional Certificate.

Original template by [John Rofrano](https://www.coursera.org/instructor/johnrofrano), Senior Technical Staff Member, DevOps Champion, @ IBM Research

## License

Licensed under the Apache License. See [LICENSE](LICENSE)

## Repository

GitHub: [MironCo/devops-capstone-project](https://github.com/MironCo/devops-capstone-project)

---

<h3 align="center">© IBM Corporation 2022. All rights reserved.</h3>
