# First-Time Deployment Guide – Rotary Taiwan Application

This document explains the steps required to deploy the application for the first time. The process involves preparing the deployment configuration on a local laptop, uploading the changes to GitHub, and then deploying the application on the deployment server using Docker Compose.

## 1. Prepare the deployment files on your local laptop

The person responsible for deployment (for example, XYZ) must have Git installed on their laptop and access to the deployment repository.

GitHub repository: [rotary-taiwan-docker-compose](https://github.com/deeptek-IT/rotary-taiwan-docker-compose)

### Step 1.1: Clone the repository

Open a terminal on the laptop and execute:

```
git clone https://github.com/deeptek-IT/rotary-taiwan-docker-compose.git
cd rotary-taiwan-docker-compose
```

This downloads the deployment configuration files from GitHub to the local laptop. The person can then review and modify the files required for deployment.

### Step 1.2: Update environment variables

Review the relevant `.env` files, such as:

- `augbackend.env`
- `dicomacceptor.env`
- Other service-specific environment files, if applicable.

Update the required environment variables according to the client's environment, such as database connection details, service endpoints, ports, and other application-specific configurations.

Important: Use the actual values provided for the target environment. Do not commit passwords, access tokens, or other sensitive credentials to a public or shared Git repository.

### Step 1.3: Update Docker image tags

Open `docker-compose.yaml` and update the Docker image tags for the services that need to be deployed.

For example, if a service currently uses an older image tag, update it to the approved release tag.

This ensures that Docker Compose deploys the intended application versions rather than unintentionally using an older release.

### Step 1.4: Review the MySQL configuration

Review `docker-compose-mysql.yaml` to confirm that the MySQL configuration is suitable for the target environment.

Generally, no changes are required unless the database configuration, storage settings, credentials, or other deployment requirements have changed.

### Step 1.5: Commit and push the changes to GitHub

After reviewing and updating the required files, execute:

```
git add .
git commit -m "Update deployment configuration"
git push
```

These commands perform the following actions:

- `git add .` — Stages the modified files for the commit. Review the staged changes first to ensure that secrets or unintended files are not included.
- `git commit` — Records the changes in the local Git repository.
- `git push` — Uploads the committed changes to GitHub.

The changes should be pushed to the agreed deployment branch. This makes the updated configuration available for the deployment server.

## 2. Prepare the deployment server

Log in to the deployment server using the authorized SSH account.

### Step 2.1: Create the project directory

Execute:

```
mkdir -p /home/<user>/projects/
cd /home/<user>/projects/
```

This creates the directory for storing the application's deployment files, if it does not already exist.

Replace `<user>` with the actual Linux username on the server.

### Step 2.2: Clone the deployment repository

Execute the following command, replacing the placeholders with the actual branch name and repository URL:

```
git clone --branch <branch-name> --single-branch <repository-url>
```

For example, using the Rotary Taiwan repository:

```
git clone --branch <branch-name> --single-branch https://github.com/deeptek-IT/rotary-taiwan-docker-compose.git
```

This downloads the deployment files for the specified branch to the server.

Note: The branch must contain the approved deployment configuration. If the repository is private, the server must also have authorized GitHub access.

### Step 2.3: Navigate to the project directory

The directory name created by the clone command will normally match the repository name:

```
cd rotary-taiwan-docker-compose
```

Confirm that the required Docker Compose files and environment files are present before proceeding.

## 3. Start the application services

### Step 3.1: Start MySQL

Execute:

```
docker compose -f docker-compose-mysql.yaml up -d
```

This command starts the services defined in `docker-compose-mysql.yaml` in detached mode, meaning they run in the background.

Wait for MySQL to initialize and verify that the database is healthy and accessible before starting dependent application services.

### Step 3.2: Start the application

Execute:

```
docker compose up -d
```

Docker Compose reads the default `docker-compose.yaml` file and starts the application services defined in it.

The `-d` option runs the containers in the background, allowing the terminal to be used for other commands.

## 4. Verify the deployment

After starting the services, verify that the containers are running:

```
docker compose ps
```

Check the MySQL services separately:

```
docker compose -f docker-compose-mysql.yaml ps
```

If any service fails to start, inspect its logs:

```
docker compose logs --tail=100
```

For a specific service, use:

```
docker compose logs --tail=100 <service-name>
```

Finally, verify the application using the configured health-check endpoints or application URL, and confirm that the application can connect to the database.

## Deployment workflow at a glance

1\. Local laptop

Clone repository, update environment variables and image tags, review configuration.

2\. GitHub

Commit and push the approved deployment configuration to the deployment branch.

3\. Deployment server

Clone the repository and navigate to the project directory.

4\. Start services

Start MySQL first, followed by the application services.

5\. Validate deployment

Verify container status, database connectivity, application health, and access.

Important considerations for the first deployment

- Ensure Docker Engine and the Docker Compose plugin are installed on the deployment server.
- Confirm that the required ports, firewall rules, and network connectivity are configured.
- Ensure persistent storage is configured correctly for MySQL and other services that require it.
- Do not run the MySQL startup command if it would unintentionally initialize a new database or conflict with an existing database setup. Verify the intended database initialization and persistence behavior first.
- For future deployments, if the repository already exists on the server, use `git pull` on the correct branch instead of cloning it again. Review the changes before restarting services.
