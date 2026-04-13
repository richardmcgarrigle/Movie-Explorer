# CI/CD Proposal for Movie-Explorer (GitHub Pro)

## Objective
Set up a GitHub Pro-native CI/CD pipeline that validates changes and produces a Docker image as the deployable artifact for self-hosted environments.

## Current-State and Future-State Fit
- Current implementation is a basic React frontend.
- The roadmap is expected to evolve to include backend/services.
- The pipeline should therefore be container-first now, so frontend and future services can share one deployment model.

## Pipeline Visual (Mermaid)
```mermaid
flowchart LR
  A[Short-lived feature branch] --> B[PR to main trunk]
  B --> C[CI: npm ci + lint + build + tests if present]
  C -->|pass| D[Merge to main]
  D --> E[Build Docker image]
  E --> F[Push image to GHCR]
  F --> G[Deploy image to self-hosted runtime]
```

## Proposed Hosting Platform
**Self-hosted runtime with Docker images** (primary path):
- Deploy from container images built in GitHub Actions
- Store images in GitHub Container Registry (GHCR)
- Works for current React app and future API/worker services

## Proposed Pipeline Design

### 1) Continuous Integration (on pull requests)
Trigger:
- `pull_request` to `main`

Steps:
1. Checkout code
2. Set up Node.js (LTS)
3. Install dependencies (`npm ci`)
4. Run lint (`npm run lint`)
5. Run build (`npm run build`)
6. Run tests (when available)

Test-step policy:
- If no test suite exists yet, keep the test step as a non-blocking/skipped step.
- Make tests required once baseline coverage is introduced.

Expected result:
- PRs must pass quality gates before merge
- CI remains trunk-based with short-lived branches

### 2) Continuous Delivery (on merge to `main`)
Trigger:
- `push` to `main`

Steps:
1. Checkout code
2. Build Docker image (tag with commit SHA and `main`)
3. Push image to GHCR
4. Deploy image to self-hosted environment (e.g., single host Docker, Docker Compose, or Kubernetes)

Recommended start for this project:
- Begin with single-host Docker Compose for lowest operational overhead.
- Move to Kubernetes only when scaling/HA requirements justify the added complexity.

Expected result:
- Every merge to `main` produces a versioned deployable image
- Deployment uses the exact built artifact from CI/CD

## Expected Cost (High-level)
Assumptions: one private repository, GitHub Pro account, low-to-moderate activity.

- **GitHub Pro**: fixed monthly account cost
- **GitHub Actions usage**:
  - Typical low-volume project: often within included minutes
  - Higher-volume or heavier Docker builds: possible additional usage charges
- **GHCR storage/egress**:
  - Small image footprint: low cost
  - Cost increases with image size, retention, and pull volume
- **Self-hosted runtime**:
  - Main variable cost (VM/server + bandwidth + backups + monitoring)
  - If orchestration is introduced, include platform/control-plane costs in estimates

Recommendation: treat self-hosted infrastructure as primary cost driver and monitor Actions/GHCR consumption monthly.

## GitHub Pro Features to Use
1. **GitHub Actions** for CI and image build/publish/deploy jobs  
2. **GitHub Container Registry (GHCR)** for versioned deployable artifacts  
3. **Environments** (`staging`, `production`) with optional required reviewers  
4. **Branch protection rules** on `main`:
   - Require pull requests
   - Require passing status checks
   - Optionally require up-to-date branch before merge  
5. **Dependabot** (optional but recommended) for dependency and container updates  
6. **CodeQL / security scanning** (optional) for security posture

## Recommended Workflow Structure
- `ci.yml`  
  - PR validation (lint/build/tests)
- `build-image.yml`  
  - Build and publish image on `main`
- `deploy-self-hosted.yml`  
  - Deploy published image to runtime

## Secrets and Configuration
- `GHCR` publish via `GITHUB_TOKEN` (or PAT if cross-repo/org policy requires)
- Deployment secrets (host credentials, SSH key, kubeconfig, etc.) stored in GitHub Environments
- Optional repository variables:
  - `NODE_VERSION` (e.g., `20`)
  - `IMAGE_NAME`
  - `DEPLOY_TARGET`

## Branching and Release Strategy
- Trunk-based development with `main` as the trunk
- Use short-lived feature branches where needed -> Pull Request -> `main`
- CI must pass before merge
- Merge to `main` triggers image build and deployment
- Rollback approach: redeploy a previous known-good image tag

## Initial Rollout Plan
1. Add `ci.yml` and verify PR checks for current React app
2. Add Dockerfile and `build-image.yml` to publish image to GHCR
3. Add `deploy-self-hosted.yml` for target runtime
4. Configure environment protection and branch protection
5. Validate end-to-end deployment with a test PR and merge

## Success Criteria
- PRs show automated lint/build (and test when present) status checks
- Merging to `main` automatically produces a versioned Docker image
- Self-hosted deployment consumes the published image artifact
- Deployment history and logs are visible in GitHub Actions/Environments
