# Splunk App CI/CD Pipeline

This project provides a comprehensive, end-to-end local environment and CI/CD pipeline for building, testing, and deploying a Splunk App. The environment is entirely containerized, simulating a real-world multi-node Splunk architecture along with a GitLab server and runner.

## Objective

The main goal of this project is to automate the deployment of a Splunk application into a multi-node cluster (Cluster Master and a Search Head Cluster) via a GitLab CI/CD pipeline. 

The pipeline ensures:
1. **Build**: The Splunk App is properly packaged into a `.tgz` artifact.
2. **Test**: The app is validated against Splunk packaging standards using `splunk-appinspect`.
3. **Deploy**: The artifact is automatically pushed and installed to the correct Splunk management nodes (Cluster Master, Search Head Deployer) using Ansible.

## Architecture & Infrastructure

The project uses Docker Compose to stand up a complete testing and CI/CD infrastructure:

- **gitlab**: A local GitLab CE instance (`http://localhost:8080`).
- **gitlab-runner**: A Dockerized runner to execute your CI/CD jobs.
- **splunk-cm** (Cluster Master): Receives Indexer apps in `etc/master-apps/` and runs `splunk apply cluster-bundle`.
- **splunk-idx** (Indexer): Clustered with the CM to verify indexer configurations.
- **splunk-shd** (Search Head Deployer): Receives Search Head apps in `etc/shcluster/apps/` and runs `splunk apply shcluster-bundle`.
- **splunk-sh1, splunk-sh2, splunk-sh3** (Search Head Cluster): A 3-node cluster managed by the Deployer.

## How It Works

### The App

The source code for the Splunk app is located in `app/my_splunk_app/`. 

To control where the app should be deployed, a `deploy.yml` file is used inside the `app/` directory:

```yaml
targets:
  - cluster_master
  - search_head_deployer
```
This tells the deployment stage which nodes to target.

### The CI/CD Pipeline (`.gitlab-ci.yml`)

The pipeline runs in three stages:

1. **build_app**: Compresses the `my_splunk_app` directory into an `app.tgz` artifact.
2. **test_app**: Uses `splunk-appinspect` to validate the app against Splunk standards. If failures or errors are found, the pipeline stops. It allows minor warnings and future-failures to pass.
3. **deploy_app**: Uses an Ansible playbook (`ansible/deploy.yml`) to read the target nodes from `deploy.yml`, securely transfer the app to the appropriate nodes, extract it, and apply the Splunk bundles.

## Getting Started

### 1. Start the Environment
Run Docker Compose to start GitLab, the Runner, and the entire Splunk cluster. Note that spinning up the Splunk nodes might take a few minutes.
```bash
docker-compose up -d
```

### 2. Configure GitLab & Runner
- Access GitLab at `http://localhost:8080`.
- Create an initial password and log in.
- Create a new project.
- Register the `gitlab-runner` to your newly created project. (You will need to pass the registration token from your GitLab project settings to the runner).

### 3. Push the Code
Initialize a git repository in this project and push it to your local GitLab server:
```bash
git init
git remote add origin http://localhost:8080/root/your-project-name.git
git add .
git commit -m "Initial commit"
git push -u origin master
```

### 4. Watch the Magic Happen
Once pushed, the GitLab runner will pick up the `.gitlab-ci.yml` file and automatically:
- Build the app artifact.
- Run `splunk-appinspect` to validate the codebase.
- Trigger Ansible to deploy the app bundle to your Dockerized Splunk Cluster Master and Search Head Deployer.

## Security & Secret Management

This project uses environment variables to secure sensitive data like Splunk passwords and cluster secrets.

Before deploying to a real environment, you must set these secure variables:

1. **Configure GitLab CI/CD**:
   In your GitLab repository settings (Settings > CI/CD > Variables), add the following masked and protected variables so the pipeline can deploy:
   - `SPLUNK_PASSWORD`: The actual plaintext Splunk password.
   - `SPLUNK_IDXC_SECRET`: The indexer cluster secret.
   - `SPLUNK_SHC_SECRET`: The search head cluster secret.

2. **Local Testing**:
   For local `docker-compose` testing, use the `.env` file (which is ignored by Git). You can add your variables there:
   ```env
   SPLUNK_PASSWORD=YourRealPassword
   SPLUNK_IDXC_SECRET=YourIdxcSecret
   SPLUNK_SHC_SECRET=YourShcSecret
   ```

## Troubleshooting

- **AppInspect Failures**: If the `test` stage fails, check the job logs. `splunk-appinspect` will output the exact issues you need to fix in your Splunk configurations (e.g., missing stanzas in `app.conf`).
- **Ansible Deployment Issues**: Verify that the Docker networks are resolving correctly and that the `deploy.yml` target strings match your Ansible inventory groups.
