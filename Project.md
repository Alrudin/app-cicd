# Splunk App CI/CD Project

## Objective
Build a comprehensive end-to-end CI/CD pipeline that:
1. Takes a Splunk App from a GitLab repository.
2. Packages it into a `.tgz` file.
3. Deploys it to respective Splunk management nodes (Cluster Master, Search Head Deployer) using a GitLab Runner.

## Core Principles (per `Instructions.md`)
- **Simplicity First**: The testing environment will use minimal, standalone Splunk Enterprise Docker containers simulating the management nodes. No extra clustering overhead unless required for deployment validation.
- **Surgical Changes**: The pipeline will only touch required Splunk directories (`etc/apps`, `etc/deployment-apps`, `etc/master-apps`, and `etc/shcluster/apps`).
- **Goal-Driven Execution**: Each phase (Build, Deploy DS, Deploy CM, Deploy SHD) will be independently verifiable.

## Architecture & Testing Environment

### Docker Compose Infrastructure
To test the pipeline locally, we will use Docker Compose to stand up:
- **gitlab**: Local GitLab instance.
- **gitlab-runner**: Dockerized runner to execute CI/CD jobs.
- **splunk-cm** (Cluster Master): Receives Indexer apps in `etc/master-apps/` and runs `splunk apply cluster-bundle`.
- **splunk-shd** (Search Head Deployer): Receives Search Head apps in `etc/shcluster/apps/` and runs `splunk apply shcluster-bundle`.
- **splunk-idx** (Indexer): Clustered with CM to verify the cluster-bundle is actively applied.
- **splunk-sh** (Search Head): Clustered with SHD to verify the shcluster-bundle is actively applied.

### App Deployment Targeting
To identify where an app should be installed, the pipeline will rely on a `deploy.yml` file placed at the root of the app repository. The CI/CD pipeline will read this file and trigger only the relevant deployment stages.

**Example `deploy.yml`:**
```yaml
targets:
  - cluster_master
  - search_head_deployer
```

### CI/CD Pipeline (`.gitlab-ci.yml`)
1. **Stage: Build**
   - Lints the app configuration (optional, using `ksconf` or Splunk AppInspect).
   - Excludes `.git` and CI/CD files.
   - Packages the app into an artifact: `my_splunk_app.tgz`.
2. **Stage: Deploy to Cluster Master**
   - Transports the `tgz` to `splunk-cm`.
   - Extracts into `etc/master-apps/`.
   - Executes Splunk CLI command to apply bundle.
4. **Stage: Deploy to Search Head Deployer**
   - Transports the `tgz` to `splunk-shd`.
   - Extracts into `etc/shcluster/apps/`.
   - Executes Splunk CLI command to push bundle.

## Execution Plan & Verification
1. **Step 1: Docker Environment Setup** -> verify: All containers are up, and Splunk/GitLab interfaces are locally accessible.
2. **Step 2: GitLab & Runner Config** -> verify: Runner is registered to a test project in GitLab.
3. **Step 3: Pipeline Authoring (Build)** -> verify: Committing a sample app generates a valid `.tgz` artifact in GitLab.
4. **Step 4: Pipeline Authoring (Deploy)** -> verify: The Runner successfully connects to the Splunk containers, copies the files, and triggers bundle applies.
