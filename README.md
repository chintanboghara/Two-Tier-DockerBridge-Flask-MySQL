# Two-Tier Docker Application with Flask and MySQL

The application features a Flask front-end that interacts with a MySQL back-end database. This setup is ideal for learning how to containerize and connect multiple services using Docker networks.

## Manual Setup with Docker Commands

### 1. Pull the Required Images

Start by downloading the necessary Docker images from Docker Hub:

```bash
docker pull chintanboghara/two-tier-dockerbridge-flask-mysql
docker pull mysql:latest
```

- The first command pulls a custom Flask application image, pre-configured for this setup.
- The second command pulls the official MySQL image, which serves as the database.

### 2. Create a Custom Bridge Network

Create a custom bridge network to enable seamless communication between the Flask and MySQL containers:

```bash
docker network create two-tier-dockerbridge-flask-mysql -d bridge
```

A custom bridge network allows containers to communicate using their container names as hostnames, simplifying connectivity in multi-container setups.

### 3. Verify the Network Creation

Confirm that the network was successfully created:

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

Look for `two-tier-dockerbridge-flask-mysql` with the `bridge` driver in the list.

### 4. Run the MySQL Container

Launch the MySQL container on the custom network and configure it with environment variables:

```bash
docker run -d --name mysql --network two-tier-dockerbridge-flask-mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=devops \
  mysql:latest
```

- `-d`: Runs the container in detached mode (in the background).
- `-e MYSQL_ROOT_PASSWORD=root`: Sets the MySQL root password to `root`.
- `-e MYSQL_DATABASE=devops`: Creates a database named `devops` upon startup.

### 5. Run the Flask Application Container

Start the Flask application container, connecting it to the same network and configuring it to communicate with MySQL:

```bash
docker run -d -p 5000:5000 --network two-tier-dockerbridge-flask-mysql \
  -e MYSQL_HOST ­­­MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=root \
  -e MYSQL_DB=devops \
  chintanboghara/two-tier-dockerbridge-flask-mysql:latest
```

- `-p 5000:5000`: Maps port 5000 on the host to port 5000 in the container, allowing access to the Flask app from the host machine.
- Environment variables (`MYSQL_HOST`, `MYSQL_USER`, etc.) configure the Flask app to connect to the MySQL container using the hostname `mysql`.

### 6. Verify the Containers Are Running

Ensure both the MySQL and Flask containers are operational:

```bash
docker ps
```

**Expected Output:** Look for both the `mysql` container and the Flask container (with a random or specified name) listed with a status of `Up`.

### 7. Inspect the Docker Network (Optional)

To understand how the containers are connected, inspect the custom network:

```bash
docker network inspect two-tier-dockerbridge-flask-mysql
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

This step is optional but useful for confirming network connectivity.

### 8. Access the MySQL Container

Interact with the MySQL database inside the container:

```bash
docker exec -it mysql mysql -u root -p
```

- This command opens a MySQL shell inside the container. Enter the password (`root`) when prompted.
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

Access the Flask app via a web browser at:

```
http://<your-server-ip>:5000
```

- Replace `<your-server-ip>` with the actual IP address of your Docker host.
- The Flask app should display a simple interface interacting with the MySQL database.

## Setup with Docker Compose

For easier management, use **Docker Compose** to define and run multi-container applications with a single configuration file (`docker-compose.yml`). This simplifies the setup process compared to manual commands.

### Prerequisites

- Ensure a `docker-compose.yml` file exists in your working directory, defining the `flask` and `mysql` services.

### Sample `docker-compose.yml`

Below is a sample configuration file for this application:

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:latest
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: devops
    networks:
      - two-tier-network

  flask:
    image: chintanboghara/two-tier-dockerbridge-flask-mysql:latest
    container_name: flask
    ports:
      - "5000:5000"
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: root
      MYSQL_DB: devops
    depends_on:
      - mysql
    networks:
      - two-tier-network

networks:
  two-tier-network:
    driver: bridge
```

- Defines `mysql` and `flask` services connected to a custom network (`two-tier-network`).
- The `depends_on` field ensures MySQL starts before Flask.

### Docker Compose Commands

#### Build and Start the Services

Start both services in detached mode:

```bash
docker-compose up -d
```

#### Stop and Remove the Services

Stop and remove containers, networks, and volumes:

```bash
docker-compose down
```

#### Rebuild Services (if needed)

Rebuild and restart services after changes:

```bash
docker-compose up -d --build
```

#### View Logs for All Services

View real-time logs:

```bash
docker-compose logs -f
```

#### Restart Specific Services

Restart individual services:

```bash
docker-compose restart flask mysql
```

**Note:** Ensure the `docker-compose.yml` file is correctly configured in your working directory before running these commands.
