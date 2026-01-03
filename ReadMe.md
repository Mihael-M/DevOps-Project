# Spring Boot → Docker → Kubernetes Delivery Pipeline (DevSecOps)

This repository demonstrates a complete, automated software delivery process: starting from a Git repository and ending with a rolling deployment to a Kubernetes cluster. The focus is on CI/CD + security gates (DevSecOps) and “pipeline as code”.

---

## What this project shows

- **Source control workflow**: issues → feature branches → pull requests
- **Continuous Integration (CI)**: build + unit tests + quality checks
- **Security gates**: SAST + dependency/image vulnerability scanning
- **Containerization**: Docker image build & push to a registry
- **Continuous Delivery (CD)**: rolling deployment to Kubernetes (mandatory requirement)
- **Everything as code**: CI workflow, Dockerfile, Kubernetes manifests, scripts
- **Documentation as part of the solution**: this README + reproducible steps

---

## Tech Stack

### Application
- **Java + Spring Boot**: REST API service
- **Maven**: build + test runner
- **PostgreSQL**: database backend (running in Kubernetes)

### Delivery / DevOps
- **GitHub**: source control, branching, pull requests
- **GitHub Actions**: CI/CD pipeline as code (YAML workflows)
- **Docker**: container image build, tag, and push
- **Kubernetes**: deployment target (Deployment/Service + rolling updates)
- **kubectl**: deployment automation from pipeline

### Security (DevSecOps)
- **SAST (Static Application Security Testing)**: scans source code for common security issues
- **Vulnerability scanning**: scans container image and/or dependencies for known CVEs
- **Secrets hygiene**: no credentials committed to git; use GitHub secrets for tokens

> Tools can vary (e.g., Semgrep for SAST, Trivy for image scanning). The key is the security gates exist and can block a deployment.

---

## Repository Structure

- `src/` – Spring Boot application source code
- `Dockerfile` – container build instructions
- `specs/` – Kubernetes manifests (Postgres + application deployment/service)
- `.github/workflows/` – CI/CD pipeline definition (GitHub Actions)
- `README.md` – documentation and demo instructions

---

## Branching Strategy

- `main`: always deployable; deployments to Kubernetes happen from here
- `feature/<short-name>`: feature branches for changes
- Pull Request is required to merge to `main`

Typical flow:
1. Create GitHub Issue
2. Create `feature/...` branch
3. Commit work + push
4. Open PR → pipeline runs CI checks
5. Merge to `main` → pipeline runs CD (deploy)

---

## CI/CD Pipeline Overview (as code)

The pipeline starts at the Git repository and enforces quality and security gates before deployment.

### CI (on pull request)
1. **Checkout code**
2. **Build & Unit Tests** (Maven)
3. **Lint / Style Check** (e.g., Checkstyle/Spotless – optional but recommended)
4. **SAST** (static security scan)
5. **Fail fast**: if tests or security checks fail, the PR cannot be merged

### CD (on merge to `main`)
1. **Build application**
2. **Build Docker image**
3. **Scan image for vulnerabilities** (CVE scan)
4. **Push image to registry**
5. **Deploy to Kubernetes** (`kubectl apply -f specs/`)
6. **Rolling update** of the application Deployment

---

## Deep Dive: SAST (Static Application Security Testing)

SAST scans the source code without running it and can detect:
- insecure patterns (e.g., command injection risks)
- hardcoded secrets (basic cases)
- unsafe deserialization patterns
- missing security best practices in common frameworks

In this project:
- SAST runs on every Pull Request.
- If a high severity rule matches, the pipeline fails and blocks merge/deploy.

During the demo you can show:
- a deliberately introduced insecure pattern
- the pipeline failing on PR
- fixing the issue
- pipeline passing and allowing deployment

---

## Running Locally (optional)

### Build + tests
```bash
./mvnw test
