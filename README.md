# Two-Tier Docker Application: Flask and MySQL

This guide provides instructions to set up a two-tier application using Docker, featuring a Flask front-end and a MySQL back-end.

## Manual Setup with Docker Commands

### 1. Pull the Required Images

Download the custom Flask application image and the latest MySQL image from Docker Hub:

```bash
docker pull chintanboghara/two-tier-dockerbridge-flask-mysql
docker pull mysql:latest
```

- The first command retrieves the Flask application image.
- The second command fetches the official MySQL image.

### 2. Create a Custom Bridge Network

Set up a bridge network to enable communication between the Flask and MySQL containers:

```bash
docker network create two-tier-dockerbridge-flask-mysql -d bridge
```

A custom bridge network allows containers to communicate using their names as hostnames.

### 3. Verify the Network Creation

Confirm the network was created successfully:

```bash
docker network ls
```

**Expected Output:**
```
NETWORK ID     NAME                                DRIVER    SCOPE
72bcbea97a2f   bridge                              bridge    local 
88e83e3dad62   host                                host      local 
a165c0a4c888   none                                null      local 
1873a4ffdf2c   two-tier-dockerbridge-flask-mysql   bridge    local
```

Look for `two-tier-dockerbridge-flask-mysql` with the `bridge` driver.

### 4. Run the MySQL Container

Launch the MySQL container on the custom network, configuring it with environment variables:

```bash
docker run -d --name mysql --network two-tier-dockerbridge-flask-mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=devops \
  mysql:latest
```

- `-e MYSQL_ROOT_PASSWORD=root`: Sets the MySQL root password to `root`.
- `-e MYSQL_DATABASE=devops`: Creates a database named `devops`.

### 5. Run the Flask Application Container

Start the Flask container, connecting it to the same network and linking it to MySQL:

```bash
docker run -d -p 5000:5000 --network two-tier-dockerbridge-flask-mysql \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=root \
  -e MYSQL_DB=devops \
  chintanboghara/two-tier-dockerbridge-flask-mysql:latest
```

- `-p 5000:5000`: Maps port 5000 on the host to port 5000 in the container.
- Environment variables configure the Flask app to connect to the MySQL container (`mysql` as the hostname).

### 6. Verify the Containers Are Running

Ensure both containers are operational:

```bash
docker ps
```

**Expected Output:** Both `mysql` and the Flask container (with a random or specified name) should appear with `Up` status.

### 7. Inspect the Docker Network

Check the network details to confirm container connectivity:

```bash
docker inspect two-tier-dockerbridge-flask-mysql
```

**Sample Output:**
```
"Containers": {
    "a418f201c2d3...": {
        "Name": "heuristic_gagarin",
        "IPv4Address": "172.18.0.3/16"
    },
    "dc5d95845626...": {
        "Name": "mysql",
        "IPv4Address": "172.18.0.2/16"
    }
}
```

This confirms both containers are on the network with assigned IP addresses.

### 8. Access the MySQL Container

Interact with the MySQL database inside the container:

```bash
docker exec -it mysql mysql -u root -p
```

- Enter the password (`root`) when prompted.
- Select the `devops` database:
  ```sql
  use devops;
  ```
- Query the `messages` table:
  ```sql
  select * from messages;
  ```

**Sample Output:**
```
+----+-----------------+
| id | message         |
+----+-----------------+
|  1 | Hello..         |
|  2 | Chintan Boghara |
|  3 |                 |
+----+-----------------+
3 rows in set (0.01 sec)
```

### 9. Access the Flask Application

The Flask app runs on port 5000. Access it via:

```
http://<your-server-ip>:5000
```

Replace `<your-server-ip>` with your Docker host’s IP address.

## Setup with Docker Compose

If a `docker-compose.yml` file is present defining `flask` and `mysql` services, use these commands to manage the application:

### Build and Start the Services

```bash
docker-compose up -d
```

### Stop and Remove the Services

```bash
docker-compose down
```

### Rebuild Services (if needed)

```bash
docker-compose up -d --build
```

### View Logs for All Services

```bash
docker-compose logs -f
```

### Restart Specific Services

```bash
docker-compose restart flask mysql
```

**Note:** Ensure a `docker-compose.yml` file exists in your working directory with the appropriate service definitions.

This setup provides a robust two-tier environment with Flask as the front-end and MySQL as the back-end, connected via a Docker bridge network.
