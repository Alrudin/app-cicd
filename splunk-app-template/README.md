# Splunk App CI/CD Template

This repository is a starter template for managing Splunk applications using GitLab CI/CD, Ansible, ksconf, and Splunk AppInspect. You can fork and clone this repository to quickly bootstrap a new Splunk app project with a fully functional deployment pipeline.

## Features

- **Automated Validation**: Code is automatically checked using `ksconf check` for syntax errors.
- **Pre-certification Testing**: Evaluates your app against `splunk-appinspect` to ensure it meets Splunk Cloud and enterprise best practices.
- **Automated Deployment**: Pushes the application bundle directly to your Search Head Cluster (SHC) and Indexer Cluster Master using Ansible.
- **Dynamic Configuration**: Easily limit deployments using `app/deploy.yml`.

## Getting Started

### 1. Clone & Rename the App

1. Clone this repository locally.
2. Rename the `app/template_app` folder to your application's actual name.
   ```bash
   mv app/template_app app/YOUR_APP_NAME
   ```
3. Update `app/YOUR_APP_NAME/default/app.conf` to reflect your app's details.

### 2. Configure Your Environment

The CI/CD pipeline expects the following variables to be defined in your GitLab CI/CD settings (or your local `.env` file for local development/testing):

- `SPLUNK_PASSWORD`: The admin password for your Splunk environment.
- `SPLUNK_SHC_TARGET_URL`: The URL of your Search Head Cluster Captain (e.g., `https://splunk-sh1:8089`).

You can copy `.env.example` to `.env` locally to configure these for testing.

### 3. Update the Ansible Inventory

Update `ansible/inventory.ini` with the actual hostnames or IP addresses of your Splunk Cluster Master and Search Head Deployer. If you're not deploying via Docker, ensure you change the `ansible_connection` to SSH and provide appropriate credentials.

### 4. Target Specific Components

By default, the pipeline deploys to the targets specified in `app/deploy.yml`. Edit this file to specify whether you want to deploy to `cluster_master`, `search_head_deployer`, or both.

```yaml
targets:
  - cluster_master
  - search_head_deployer
```

### 5. Commit and Push

Once you've customized the app and configured your variables, simply commit your changes and push to GitLab. The `.gitlab-ci.yml` pipeline will automatically package your app, test it, and deploy it to the specified targets.
