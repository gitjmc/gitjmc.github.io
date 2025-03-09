---
title: Dokerize MariaDB with docker secrets 
author:
  name: 
  link: 
date: 2025-03-01 07:35:00 -0400
categories: [DEV]
tags: [mariadb, dbeaver, mysql, docker, compose, secrets]
render_with_liquid: false
---

## Docker compose MariaDB

I’ll provide you with a `docker-compose.yml` example for MariaDB that incorporates secrets for secure handling of sensitive data like the root password and database user credentials. Docker Compose supports secrets, which are a secure way to manage sensitive information without hardcoding them into the configuration file.

Here’s a sample `docker-compose.yml` file with MariaDB and secrets:

```yaml
version: '3.8'

services:
  mariadb:
    image: mariadb:latest
    restart: unless-stopped
    environment:
      - MYSQL_ROOT_PASSWORD_FILE=/run/secrets/mysql_root_password
      - MYSQL_DATABASE=my_database
      - MYSQL_USER=my_user
      - MYSQL_PASSWORD_FILE=/run/secrets/mysql_user_password
    volumes:
      - mariadb_data:/var/lib/mysql
    secrets:
      - mysql_root_password
      - mysql_user_password
    ports:
      - "3306:3306"

volumes:
  mariadb_data:

secrets:
  mysql_root_password:
    file: ./secrets/mysql_root_password.txt
  mysql_user_password:
    file: ./secrets/mysql_user_password.txt
```

### How to Use This

1. **Directory Structure**: Create a directory for your project, e.g., `mariadb-docker`, and inside it:
   - Place the `docker-compose.yml` file.
   - Create a `secrets` subdirectory.

2. **Create Secret Files**:
   - In the `secrets` subdirectory, create two files:
     - `mysql_root_password.txt`: Write your desired MariaDB root password (e.g., `mysecretrootpass`).
     - `mysql_user_password.txt`: Write your desired database user password (e.g., `mysecretuserpass`).
   - Ensure these files contain only the password (no extra spaces or newlines).

   Example structure:
   ```
   mariadb-docker/
   ├── docker-compose.yml
   └── secrets/
       ├── mysql_root_password.txt
       └── mysql_user_password.txt
   ```

3. **Run Docker Compose**:
   - Open a terminal in the `mariadb-docker` directory.
   - Run:
     ```bash
     docker-compose up -d
     ```
   - This starts MariaDB in detached mode (`-d`).

4. **Verify**:
   - Check the container is running: `docker ps`.
   - Connect to MariaDB using a client (e.g., `mysql` CLI, DBeaver) with:
     - Host: `localhost`
     - Port: `3306`
     - User: `my_user` (or `root` for admin access)
     - Password: From the respective secret file.

### Explanation
- **Secrets**: The `mysql_root_password` and `mysql_user_password` are stored in files and mounted into the container via Docker secrets. MariaDB reads them from `/run/secrets/`.
- **Environment Variables**: 
  - `MYSQL_ROOT_PASSWORD_FILE` and `MYSQL_PASSWORD_FILE` tell MariaDB to use the secrets.
  - `MYSQL_DATABASE` creates a database named `my_database`.
  - `MYSQL_USER` creates a user named `my_user`.
- **Volumes**: Persistent data is stored in the `mariadb_data` volume.
- **Ports**: Maps port `3306` on your host to the container.

### Notes
- **Security**: Keep the `secrets` directory out of version control (e.g., add it to `.gitignore`). In production, consider using a secret management tool like Docker Swarm secrets or a vault solution.
- **Customization**: Adjust the database name, user, or port mappings as needed.
- **MariaDB Version**: `mariadb:latest` uses the latest version; you can pin it to a specific version (e.g., `mariadb:10.11`).

## Troubleshoting to connect to DBeaver

If you're receiving an "Access denied for user 'root'" error when trying to connect to MariaDB via DBeaver, it’s likely due to authentication issues, misconfigured credentials, or network settings. Let’s troubleshoot and set up the connection step-by-step using the `docker-compose.yml` setup I provided earlier.

### Prerequisites
- Ensure your MariaDB container is running (from the `docker-compose up -d` command).
- You have DBeaver installed on your machine.
- You know the credentials from your secrets files (`mysql_root_password.txt` and `mysql_user_password.txt`).

### Steps to Connect to MariaDB with DBeaver

1. **Verify MariaDB is Running**
   - Run `docker ps` to confirm the MariaDB container is up. You should see something like:
     ```
     CONTAINER ID   IMAGE            COMMAND                  PORTS                  NAMES
     xyz123         mariadb:latest   "docker-entrypoint..."   0.0.0.0:3306->3306/tcp mariadb-docker_mariadb_1
     ```
   - If it’s not running, start it with `docker-compose up -d` from the directory containing your `docker-compose.yml`.

2. **Check Credentials**
   - From the `docker-compose.yml` I provided:
     - **Root User**: `root` with the password from `secrets/mysql_root_password.txt` (e.g., `mysecretrootpass`).
     - **Custom User**: `my_user` with the password from `secrets/mysql_user_password.txt` (e.g., `mysecretuserpass`).
   - Open these files to confirm the exact passwords you set.

3. **Open DBeaver and Create a New Connection**
   - Launch DBeaver.
   - Click **Database** > **New Database Connection** (or the "+" icon in the Database Navigator).
   - Select **MariaDB** from the list and click **Next**.

4. **Configure Connection Settings**
   - **Host**: `localhost` (since the port is mapped to your host via `3306:3306` in Docker Compose).
   - **Port**: `3306` (default MariaDB port, mapped in the `docker-compose.yml`).
   - **Database**: `my_database` (from the `MYSQL_DATABASE` environment variable; optional for root).
   - **Username**: Try `root` first (or `my_user` if you prefer the custom user).
   - **Password**: Enter the password from `mysql_root_password.txt` (e.g., `mysecretrootpass`) for `root`, or from `mysql_user_password.txt` for `my_user`.
   - Example:
     - Host: `localhost`
     - Port: `3306`
     - Username: `root`
     - Password: `mysecretrootpass`

5. **Test the Connection**
   - Click **Test Connection** in DBeaver.
   - If it works, you’ll see a success message. If not, you’ll get an error (e.g., "Access denied").

6. **Troubleshooting "Access Denied"**
   If you get "Access denied for user 'root'@'localhost'":
   - **Wrong Password**: Double-check the password from `mysql_root_password.txt`. Copy-paste it to avoid typos.
   - **Root Login Restrictions**: MariaDB in Docker might restrict root login from outside the container by default. Let’s try the custom user instead:
     - Username: `my_user`
     - Password: From `mysql_user_password.txt` (e.g., `mysecretuserpass`).
     - Test again.
   - **Network Issue**: Ensure you’re connecting to `localhost:3306` and no firewall is blocking the port. Run `telnet localhost 3306` (or equivalent) to check connectivity.
   - **Container Logs**: Check MariaDB logs for clues:
     ```bash
     docker logs <container_name>
     ```
     (Replace `<container_name>` with the name from `docker ps`, e.g., `mariadb-docker_mariadb_1`.)

7. **Fix Root Access (if Needed)**
   If `root` still fails but `my_user` works, and you need root access:
   - Access the container:
     ```bash
     docker exec -it <container_name> bash
     ```
   - Log in to MariaDB inside the container:
     ```bash
     mariadb -u root -p
     ```
     Enter the root password from `mysql_root_password.txt`.
   - Grant root access from any host (less secure, for testing only):
     ```sql
     GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'mysecretrootpass';
     FLUSH PRIVILEGES;
     ```
   - Exit and retry in DBeaver with `root`.

8. **Finalize**
   - Once the connection works, click **Finish** in DBeaver to save it.
   - You should now see `my_database` (or other databases if using root) in the navigator.

### Common Issues
- **"Host '172.x.x.x' is not allowed to connect"**: This happens if MariaDB rejects connections from your host. Using `my_user` should work since it’s created with broader permissions, or adjust root as above.
- **Port Conflict**: If `3306` is already in use on your machine, change the host mapping in `docker-compose.yml` (e.g., `3307:3306`) and use `3307` in DBeaver.


