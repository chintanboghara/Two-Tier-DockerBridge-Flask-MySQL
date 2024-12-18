The command `docker pull chintanboghara/two-tier-dockerbridge-flask-mysql` is used to download an image named `two-tier-dockerbridge-flask-mysql` from Docker Hub, under the repository `chintanboghara`.

### **Pull the Image**:
```bash
docker pull chintanboghara/two-tier-dockerbridge-flask-mysql
```
### **Pull the Latest MySQL Image**:
```bash
docker pull mysql:latest
```

## Steps to Set Up the Environment

### 1. Create a Docker Bridge Network

Create a custom bridge network to allow communication between the Flask and MySQL containers:

```bash
docker network create two-tier-dockerbridge-flask-mysql -d bridge
```

### 2. Verify the Network Creation

List all available Docker networks to ensure the custom bridge network was created:

```bash
docker network ls
```

You should see an output like this:

```bash
NETWORK ID     NAME                                DRIVER    SCOPE
72bcbea97a2f   bridge                              bridge    local 
88e83e3dad62   host                                host      local 
a165c0a4c888   none                                null      local 
1873a4ffdf2c   two-tier-dockerbridge-flask-mysql   bridge    local
```

### 3. Run the MySQL Container

Run the MySQL container and link it to the created bridge network. The environment variables set the root password and database name:

```bash
docker run -d --name mysql --network two-tier-dockerbridge-flask-mysql -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=devops mysql:latest
```

### 4. Run the Flask Application Container

Run the Flask application, linking it to the same network. Set the MySQL connection parameters through environment variables:

```bash
docker run -d -p 5000:5000 --network two-tier-dockerbridge-flask-mysql \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=root \
  -e MYSQL_DB=devops \
  two-tier-dockerbridge-flask-mysql:latest
```

### 5. Verify the Containers Are Running

Check that both containers (Flask and MySQL) are running:

```bash
docker ps
```

### 6. Inspect the Docker Network

Check the details of the network, including the IP addresses of the containers:

```bash
docker inspect two-tier-dockerbridge-flask-mysql
```

The output should display container details similar to this:

```bash
"Containers": {
    "a418f201c2d3ad2805932d9c8d6d07ca8dd20e5d955effda0b7206ecd6950682": {
        "Name": "heuristic_gagarin",
        "IPv4Address": "172.18.0.3/16"
    },
    "dc5d95845626193f350ef11ed3e9ccac67cae44fbc0fefca4c64a2693c7b2027": {
        "Name": "mysql",
        "IPv4Address": "172.18.0.2/16"
    }
}
```

### 7. Access the MySQL Container

Execute the following command to access the MySQL container and interact with the database:

```bash
docker exec -it mysql mysql -u root -p
```

Enter the MySQL root password (`root` in this case) and then select the `devops` database:

```bash
mysql> use devops;
```

You can view the data from the `messages` table:

```bash
mysql> select * from messages;
```

Example output:

```bash
+----+-----------------+
| id | message         |
+----+-----------------+
|  1 | Hello..         |
|  2 | Chintan Boghara |
|  3 |                 |
+----+-----------------+
3 rows in set (0.01 sec)
```

This setup provides a simple two-tier environment with Flask as the front-end application and MySQL as the back-end database, all within Docker containers connected via a bridge network.

- The Flask app is running on port `5000`. Access it via `http://<your-server-ip>:5000`.

## **Docker Compose Commands**

### Build and Start the Services:

```bash
docker-compose up -d
```

### Stop and Remove the Services:

```bash
docker-compose down
```

### Rebuild Services (if needed):

```bash
docker-compose up -d --build
```

### View Logs for All Services:

```bash
docker-compose logs -f
```

### Restart Specific Services:

```bash
docker-compose restart flask mysql
```
