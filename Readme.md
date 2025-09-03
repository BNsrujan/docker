
```bash
#!/bin/bash

# List all running containers
docker ps

# List all Docker images (including intermediate ones)
docker image ls --all
docker image ls -a

# List all Docker containers (including stopped ones)
docker container list --all

# Enter into a running container
docker exec -it <container_id> bash

# Build a Docker image from a Dockerfile in the current directory
sudo docker build . -t <image_name>

# Run the Docker image with interactive terminal
sudo docker run -it <image_name> bash

# Run the Docker image and map ports
sudo docker run -p 8000:8000 <image_name>

# Log into DockerHub
sudo docker login

# Push the image to a repository on DockerHub
sudo docker push <user_name>/<repo_name>

# Create a Docker volume
sudo docker create volume <volume_name>

# Run a container with a volume and port mapping
sudo docker run -v <volume_name>:<container_directory> -p 8000:8000 <docker_image_id>
sudo docker run -v <volume_name>:<container_directory> -p 8000:8000 <docker_image_id> bash

# Run a MongoDB container and attach it to a network
sudo docker run -v volume_database:/data/db --name mongo_database --network mongo_network -p 27017:27017 -d mongo

# Run a Node.js app container and connect it to the MongoDB container
sudo docker run -p 3000:3000 --network mongo_network -e MONGO_URL=mongodb://mongo_database:27017/todos node_app

# List all Docker volumes
sudo docker volume ls

# Inspect a Docker volume
sudo docker volume inspect <volume_name>

# Remove a specific volume
sudo docker volume rm <volume_name>

# Prune (remove) all unused volumes
sudo docker volume prune

# Create a named volume
sudo docker volume create --name <volume_name>

# Copy files from a container to the host
sudo docker cp <container_id>:<container_directory> <host_directory>

# Create a Docker network
sudo docker create network <network_name>

# List all Docker networks
sudo docker network ls

# Inspect a specific Docker network
sudo docker network inspect <network_name>

# Connect a container to a specific network
sudo docker network connect <network_name> <container_id>

# Remove a Docker container by ID
sudo docker kill <container_id>

# Remove a Docker volume by name
sudo docker volume rm volume_database

# Remove all Docker volumes
sudo docker volume rm $(sudo docker volume ls -q)

# List all containers again
sudo docker ps -a

# List only running containers
sudo docker ps

# Remove all stopped containers
sudo docker rm $(sudo docker ps -a -q)

#stop the container before removing 
docker stop $(docker ps -a -q)

#stop the container using docker compose file 
docker-compose down -v  #-v will remove the docker volume atached to it 
docker-compose up -d # -d detached mode remove 

```

The commands **`sudo docker kill <container_id>`** and **`sudo docker rm $(sudo docker ps -a -q)`** perform different functions. Let’s break them down:

---

## **1. `sudo docker kill <container_id>`**

This command **forcibly stops** a running container **immediately**.

### 🔹 What it does:

- It **kills** (stops immediately) the container by **sending a `SIGKILL` signal**.
- Unlike `docker stop`, which allows a graceful shutdown, `docker kill` **terminates the process immediately**.
- The container will still **exist** but in a **stopped state** (you can see it with `docker ps -a`).

### ✅ Example:

```bash
sudo docker kill e85ae05f6cf9
```

This immediately kills the container with ID `e85ae05f6cf9`.

---

## **2. `sudo docker rm $(sudo docker ps -a -q)`**

This command **removes all stopped containers**.

### 🔹 What it does:

- **`docker ps -a -q`**: Lists **all** container IDs (both running and stopped).
- **`docker rm`**: Deletes the containers listed.

### ✅ Example:

```bash

sudo docker rm $(sudo docker ps -a -q
```

This removes **all** stopped containers from your system.

🔸 **Important:** If a container is **running**, `docker rm` won’t remove it unless you stop it first. You can use:

```bash
sudo docker rm -f $(sudo docker ps -a -q)
```

The `-f` flag forcefully stops and removes all containers.

---

### **⚡ Key Differences**

| Command | Action |
| --- | --- |
| `docker kill <container_id>` | **Immediately stops** a specific running container (but does NOT remove it). |
| `docker rm $(docker ps -a -q)` | **Deletes all stopped containers** from the system. |

---

### **🛠️ When to Use Which?**

- Use **`docker kill`** when you need to **immediately stop** a specific running container.
- Use **`docker rm`** when you want to **clean up** stopped containers to free up space.

### **1️⃣ If You Kill (`docker kill`) and Restart (`docker start`) the Container**

🔹 **Data is persistent** as long as:

- The container is restarted **without being removed**.
- The data is stored **inside the container's filesystem** (but this is not recommended).

✅ Example:

```bash
bash
CopyEdit
sudo docker kill <container_id>   # Kills the container
sudo docker start <container_id>  # Restarts the container (data remains)

```

➡️ **Data will still be there** if the container was using its internal storage.

### example :

```docker
version: '3.9'

services:
  postgres:
    image: postgres:16.3
    container_name: postgres
    restart: always 
    environment:
      POSTGRES_DB: tasks
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: adminPassword
    ports:
      - "5432:5432" # outside port : inside:port
    volumes:
      - postgres_data:/var/lib/postgresql/data

  pgadmin:
    image: dpage/pgadmin4
    container_name: pgadmin
    restart: always
    depends_on:
      - postgres
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com  
      PGADMIN_DEFAULT_PASSWORD: adminPassword  
    ports:
      - "5050:80"

volumes:
  postgres_data:
    driver: local

```

In **pgAdmin**, instead of using `localhost`, use the **service name** `postgres`.
both containers are in the same Docker network, they communicate via container names.
