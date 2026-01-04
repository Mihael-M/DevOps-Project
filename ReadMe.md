# DevOps-Project — Automated Delivery Pipeline (CI/CD + Security + Kubernetes)

A small Spring Boot application used to demonstrate a **complete automated software delivery process**:
- **CI** on Pull Requests (tests + SAST)
- **CD** on pushes to `main` (build image + vulnerability scan + push to registry + deploy to Kubernetes)

The goal is not the app complexity, but the **DevOps process** around it.

---

## What this project covers (7+ course topics)

1. **Source control** — GitHub repository as the single source of truth  
2. **Branching strategy / Collaboration** — feature branches + Pull Requests  
3. **Building Pipelines** — GitHub Actions workflows as code  
4. **Continuous Integration (CI)** — automated tests on PR  
5. **Security**  
   - **SAST** with Semgrep (code/config scanning)
   - **Image vulnerability scanning** with Trivy (CD gate)
6. **Docker** — container image build via Dockerfile  
7. **Kubernetes** — Deployment + Service + rolling updates  
8. **Documentation** — this README explains the process & commands

---

## High-level flow

### CI (Pull Request → quality & security gate)
- Checkout
- Run unit tests (`./mvnw test`)
- Run **Semgrep SAST** (OWASP Top Ten ruleset)

If CI fails → PR should not be merged.

### CD (Push to `main` → build & deploy)
- Checkout
- Login to **GHCR**
- Build Docker image
- Run **Trivy** scan (**fails** on HIGH/CRITICAL)
- Push image to GHCR
- Apply Kubernetes manifests
- Rolling update on the deployment

---

## Diagram (Pipeline)

```mermaid
flowchart LR
  A[Developer pushes to feature/*] --> B[Pull Request]
  B --> C[CI: Tests]
  C --> D[CI: Semgrep SAST]
  D -->|green| E[Merge to main]
  D -->|red| X[Fix issues]

  E --> F[CD: Docker build]
  F --> G[CD: Trivy scan]
  G -->|green| H[Push image to GHCR]
  G -->|red| Y[Upgrade deps / fix vulnerabilities]

  H --> I[Deploy to Kubernetes]
  I --> J[Rolling update]


  flowchart TB
  subgraph K8s[Kubernetes Cluster (minikube)]
    Dp[Deployment: spring-demo] --> Pod[Pod: spring-demo]
    Svc[Service: spring-demo (ClusterIP)] --> Pod
  end

  Dev[Developer machine] -->|kubectl port-forward| Svc
  Pod -->|HTTP 8080| App[Spring Boot API]

  Repository structure
	•	spring-boot-project/ — Spring Boot application
	•	Dockerfile — builds the app image
	•	k8s/
	•	deployment.yml — Kubernetes Deployment
	•	service.yml — Kubernetes Service (ClusterIP)
	•	.github/workflows/
	•	ci.yml — CI pipeline (PR)
	•	cd.yml — CD pipeline (push to main)

    Security notes (what we enforce)

SAST (Semgrep)

Semgrep scans source and configuration (including Kubernetes YAML).
Example: it flagged missing container securityContext.allowPrivilegeEscalation: false, which was fixed in the deployment manifest.

Trivy (Image scanning)

The CD pipeline runs Trivy on the built image and blocks deployment if it finds HIGH/CRITICAL vulnerabilities.

⸻

How to run locally (no Kubernetes)

From repo root:

cd spring-boot-project
./mvnw clean test
./mvnw clean package -DskipTests
java -jar target/*.jar

Then open:
	•	http://localhost:8080/api/tutorials

Docker (local)

From spring-boot-project/ (where the Dockerfile is):

docker build -t devops-project:local .
docker run --rm -p 8080:8080 devops-project:local

Test:

curl -i http://localhost:8080/api/tutorials

Kubernetes (minikube)

Start cluster:

minikube start
kubectl get nodes

Apply manifests:

kubectl apply -f spring-boot-project/k8s
kubectl rollout status deployment/spring-demo
kubectl get pods -l app=spring-demo
kubectl get svc spring-demo

Access the service locally:

kubectl port-forward svc/spring-demo 8080:8080

Then open:
	•	http://localhost:8080/api/tutorials

Container registry (GHCR)

Images are published to GitHub Container Registry:

ghcr.io/mihael-m/devops-project:sha-<commit_sha>

The CD pipeline tags each image with the commit SHA for traceability.