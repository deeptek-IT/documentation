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

### Step 1.2: Updating SSL files

Put SSL files (augmento.crt, augmento.key) in the folder  "volumes/nginx/" 
### Step 1.3: Update environment variables

Review the relevant `.env` files, such as:

- `augbackend.env`
- `dicomacceptor.env`
- Other service-specific environment files, if applicable.

Update the required environment variables according to the client's environment, such as database connection details, service endpoints, ports, and other application-specific configurations.

Important: Use the actual values provided for the target environment. Do not commit passwords, access tokens, or other sensitive credentials to a public or shared Git repository.

### Step 1.4: Update Docker image tags

Open `docker-compose.yaml` and update the Docker image tags for the services that need to be deployed.

For example, if a service currently uses an older image tag, update it to the approved release tag.

This ensures that Docker Compose deploys the intended application versions rather than unintentionally using an older release.

### Step 1.5: Review the MySQL configuration

Review `docker-compose-mysql.yaml` to confirm that the MySQL configuration is suitable for the target environment.

Generally, no changes are required unless the database configuration, storage settings, credentials, or other deployment requirements have changed.

### Step 1.6: Commit and push the changes to GitHub

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



## 5. Execute Database Queries (This step is not mandatory step, should be used if needed)

This step explains how to execute the database queries provided with the application release. These queries must be executed against the correct database after the application’s database configuration has been verified.

### Step 5.1: Retrieve the database credentials

On the deployment server, navigate to the directory containing the MySQL environment file and review its contents:

```
cat mysql.env
```

Identify the configured MySQL username, password, and other relevant connection details.

Security note: The file may contain sensitive credentials. Do not share its contents in screenshots, logs, or public channels.

### Step 5.2: Access the MySQL container

Execute the following command to open a shell inside the MySQL container:

```
docker exec -it dt_mysql bash
```

This opens an interactive terminal inside the running container named `dt_mysql`.

### Step 5.3: Log in to MySQL

From inside the container, execute:

```
mysql -u <user> -p
```

Replace `<user>` with the username obtained from `mysql.env`. When prompted, enter the corresponding password.

Using `-p` without specifying the password directly is recommended because it avoids exposing the password in the command itself.

### Step 5.4: Select the application database

Once logged in to MySQL, select the database on which the release queries must be executed:

```
USE <database_name>;
```

Replace `<database_name>` with the correct application database name.

Verify the selected database before proceeding:

```
SELECT DATABASE();
```

### Step 5.5: Locate the release SQL file

Open a new terminal session on the deployment server, leaving the existing MySQL session open.

Navigate to the application deployment directory:

```
cd /home/user/projects/deeptek-augmento-lite
```

The release-specific database queries are stored in the `db-queries` directory. Locate the SQL file corresponding to the release being deployed.

For example:

```
ls db-queries/
```

Then display the relevant SQL file:

```
cat db-queries/release-V<version>-queries.sql
```

Replace `<version>` with the actual release version.

### Step 5.6: Execute the release queries

Copy the SQL statements displayed in the second terminal and paste them into the existing MySQL terminal, where the application database has already been selected.

Execute the statements and review the output for errors. Some queries may return results, while others may report the number of affected rows.

Important: Before executing the queries, confirm that the SQL file corresponds to the intended release and database. Review any `UPDATE`, `DELETE`, `DROP`, or schema-changing statements carefully. Take a database backup before executing release queries, as appropriate for the deployment procedure.

### Alternative: Execute the SQL file directly

Instead of copying and pasting the queries manually, you can execute the file directly from the deployment directory:

```
docker exec -i dt_mysql mysql -u <user> -p <database_name> < db-queries/release-V<version>-queries.sql
```

This command prompts for the MySQL password and passes the SQL file directly to the MySQL client inside the container. It avoids manual copying and reduces the risk of missing statements.

Use the method approved for the deployment process, and verify that the execution completed successfully before proceeding.
