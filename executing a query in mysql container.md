## 5. Execute Database Queries

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
