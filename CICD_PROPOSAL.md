# CI/CD Proposal for Movie-Explorer (GitHub Pro)

## Objective
Set up a GitHub Pro-native CI/CD pipeline that automatically validates code changes and deploys the application for hosting with minimal operational overhead.

## Pipeline Visual (Mermaid)
```mermaid
flowchart LR
  A[Short-lived feature branch] --> B[Open PR to main]
  B --> C[CI: npm ci + lint + build]
  C -->|pass| D[Merge to main trunk]
  D --> E[CD: build production artifact]
  E --> F[Deploy to GitHub Pages]
```

## Proposed Hosting Platform
**GitHub Pages** (best fit for a static frontend app):
- Native GitHub integration
- Zero external infrastructure
- Automated deployments from GitHub Actions
- Custom domain support (optional)

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

Expected result:
- PRs must pass lint/build before merge
- Faster feedback and consistent quality gates

### 2) Continuous Deployment (on merge to main)
Trigger:
- `push` to `main`

Steps:
1. Checkout code
2. Set up Node.js (LTS)
3. Install dependencies (`npm ci`)
4. Build production assets (`npm run build`)
5. Upload artifact
6. Deploy to GitHub Pages using Actions deployment workflow

Expected result:
- Every merge to `main` produces an automatic production deployment

## GitHub Pro Features to Use
1. **GitHub Actions** for CI and deployment jobs  
2. **GitHub Pages** for application hosting  
3. **Environments** (`production`) with optional required reviewers for deploy approval  
4. **Branch protection rules** on `main`:
   - Require pull requests
   - Require passing status checks (CI workflow)
   - Optionally require up-to-date branch before merge  
5. **Dependabot** (optional but recommended) for dependency updates  
6. **CodeQL / security scanning** (optional) for security posture

## Recommended Workflow Structure
- `ci.yml`  
  - Runs lint/build checks for PR validation
- `deploy-pages.yml`  
  - Builds and deploys on `main`

## Secrets and Configuration
- No deployment secrets required for standard GitHub Pages deployment using `GITHUB_TOKEN`
- Optional repository variables:
  - `NODE_VERSION` (e.g., `20`)

## Branching and Release Strategy
- Trunk-based development with `main` as the trunk
- Use short-lived feature branches where needed -> Pull Request -> `main`
- CI must pass before merge
- Merge to `main` triggers deployment
- Rollback approach: revert commit on `main` and redeploy automatically

## Initial Rollout Plan
1. Add CI workflow file and verify PR checks
2. Add GitHub Pages deployment workflow
3. Enable Pages source as **GitHub Actions**
4. Configure branch protection for `main`
5. Validate end-to-end deployment with a test PR and merge

## Success Criteria
- PRs show automated lint/build status checks
- Merging to `main` automatically updates the hosted app
- Deployment history and logs are visible in GitHub Actions/Environments
