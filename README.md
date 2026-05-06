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

This project uses Ansible Vault and environment variables to secure sensitive data like Splunk passwords and cluster secrets.

Before deploying to a real environment, you must replace the temporary encrypted strings with your actual secure passwords:

1. **Create a Vault Password File**:
   Create a `.vault_pass` file at the root of the repository containing your secure vault password. This file is ignored by Git.
   ```bash
   echo "YourSecureVaultPassword" > .vault_pass
   ```

2. **Encrypt Your Secrets**:
   Use `ansible-vault encrypt_string` to encrypt your real Splunk password and secrets, then paste the generated blocks into `default.yml` replacing the existing `!vault | ...` entries.
   ```bash
   ansible-vault encrypt_string --vault-password-file .vault_pass 'YourRealPassword'
   ```

3. **Configure GitLab CI/CD**:
   In your GitLab repository settings (Settings > CI/CD > Variables), add the following masked variables so the pipeline can decrypt and deploy:
   - `ANSIBLE_VAULT_PASSWORD`: The contents of your `.vault_pass` file.
   - `SPLUNK_PASSWORD`: The actual plaintext Splunk password (injected into `docker-compose` by the pipeline).

4. **Local Testing**:
   For local `docker-compose` testing, copy `.env.example` to `.env` and set `SPLUNK_PASSWORD=YourRealPassword`.

## Troubleshooting

- **AppInspect Failures**: If the `test` stage fails, check the job logs. `splunk-appinspect` will output the exact issues you need to fix in your Splunk configurations (e.g., missing stanzas in `app.conf`).
- **Ansible Deployment Issues**: Verify that the Docker networks are resolving correctly and that the `deploy.yml` target strings match your Ansible inventory groups.
