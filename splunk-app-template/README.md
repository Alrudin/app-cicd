# Splunk App CI/CD Template

This repository is a starter template for managing Splunk applications using GitLab CI/CD, Ansible, ksconf, and Splunk AppInspect. You can fork and clone this repository to quickly bootstrap a new Splunk app project with a fully functional deployment pipeline.

## Features

- **Automated Validation**: Code is automatically checked using `ksconf check` for syntax errors.
- **Pre-certification Testing**: Evaluates your app against `splunk-appinspect` to ensure it meets Splunk Cloud and enterprise best practices.
- **Automated Deployment**: Pushes the application bundle directly to your Search Head Cluster (SHC) and Indexer Cluster Master using Ansible.
- **Dynamic Configuration**: Easily limit deployments using `app/deploy.yml`.

## Getting Started & Developer Workflow

### 1. Fork the Repository in GitLab
1. Navigate to this repository in your GitLab instance.
2. Click the **Fork** button in the top right corner.
3. Select your personal namespace or target group to create your own copy of the template.

### 2. Clone Your Fork Locally
Open your terminal and clone your newly forked repository:
```bash
git clone https://gitlab.example.com/YOUR_USERNAME/splunk-app-template.git
cd splunk-app-template
```

### 3. Create a New Branch
Always create a new branch for your app modifications:
```bash
git checkout -b feature/my-new-splunk-app
```

### 4. Rename and Configure the App
1. Rename the `app/template_app` folder to your application's actual name.
   ```bash
   mv app/template_app app/YOUR_APP_NAME
   ```
2. Update `app/YOUR_APP_NAME/default/app.conf` to reflect your app's details.
3. Edit `app/deploy.yml` to specify whether you want to deploy to `cluster_master`, `search_head_deployer`, or both.
4. Update `ansible/inventory.ini` with your actual Splunk environment hostnames or IPs.

### 5. Add, Commit, and Push Your Changes
Once you've customized the app and configured your targets, stage and commit your changes:
```bash
# Stage all changes
git add .

# Commit with a descriptive message
git commit -m "Initial setup for my new Splunk app"

# Push the branch to your GitLab fork
git push -u origin feature/my-new-splunk-app
```

### 6. Create a Merge Request
1. Go to your repository in the GitLab UI. A banner will usually appear prompting you to **Create merge request**.
2. Alternatively, navigate to **Merge requests** and click **New merge request**.
3. Select `feature/my-new-splunk-app` as the source branch and `main` as the target branch.
4. Fill in the title and description for your changes.
5. In the **Reviewers** field on the right sidebar, select a team member to review your code.
6. Click **Create merge request**.

Once the reviewer approves and merges your code into the `main` branch, the `.gitlab-ci.yml` pipeline will automatically trigger to package, test, and deploy your app.

## CI/CD Environment Variables
The automated pipeline expects the following variables to be defined in your GitLab project's CI/CD settings (Settings > CI/CD > Variables):
- `SPLUNK_PASSWORD`: The admin password for your Splunk environment.
- `SPLUNK_SHC_TARGET_URL`: The URL of your Search Head Cluster Captain (e.g., `https://splunk-sh1:8089`).

*(Note: You can copy `.env.example` to `.env` to configure these variables for local testing.)*
