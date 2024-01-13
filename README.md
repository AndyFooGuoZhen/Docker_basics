## Resource : Link: https://www.youtube.com/watch?v=3xDAU5cvi5E&list=PLxAp0ELKVyrnjVDfF3tWhhW7aAtgUsl9N&index=1

# Docker commands
Checking images on our pc 
```
docker image ls
```

### Creating a docker image
```
Docker build -t <imagename> .
```

### Deleting a docker image
```
docker image rm <image id>
```

### Check containiers on local machine
```
docker ps
```

### Creating and run a docker image in a container
```
docker run -d --name <containerName> <imagename>
```

### Running a docker image in a container with port forwarding
```
docker run -d -p 3000:3000 --name <containerName> <imageName>
```

anything send to our local machine at port 3000 will be sent to container's port 3000
localhost:3000 is like an outside host as well, it gets sent to port 3000 of container

React app example:
```
docker run -d -p 3000:3000 <imagename>
```

### Running a docker image with live updates (reflecting local changes)
Bind volume ( syncs local directory to directory in container)
```
docker run -d -p 3000:3000 --name <containerName> -v $(pwd)/src:/app/src <imagename>
```

- Bind volume also sync files from containiers
- any files added to container will be added to local
- For windows, live update command may require:
  ```
  docker run -e CHOKIDAR_USEPOLLING=true -d -p 3000:3000 --name <containerName> -v $(pwd)/src:/app/src <imagename>
  ```
  
### Removing a container
```
docker rm <containerName> -f
```

### Stopping a container
```
docker stop <containerName>
```

### Looking into files of our container (with bash commands)
NOTE: alpine ver of node wont support bash
```
docker exec -it <containerName> bash
```

Once the command is ran, you can use linux command here. Type "exit" to quit container.


### Creation of docker repo 

1. login
    docker login and enter credentials

2. docker tag <imagename>:<imagetag>  <username>/<reponame>

3. docker push <username>/<reponame>

### Sample Dockerfile

```
FROM node:alpine 
WORKDIR /app
COPY package*.json ./   # copy package.json to current directory in container
                        #optimization technique, only if package json is changed, npm install will run
                        #else, npm install won't run
RUN npm install
COPY . .
CMD ["npm", "start"]
```

### Workdir in detail
Example dockerfile

```
WORKDIR /app
COPY aws_tutorial/package*.json ./
RUN npm install --verbose
EXPOSE 3000
COPY aws_tutorial ./
```

1. Sets workdir as /app
2. By running docker exec -it <container-id> sh, and ls, you can see your container directory in /app
3. COPY aws_tutorial/package*.json ./ --> copy everything with the name package.json into /app
4. aws_tutorial ./ --> copy everything in aws_tutorial folder into /app

# Docker compose 
Docker Compose is a tool that allows you to define and manage multi-container Docker applications. It provides a simple way to define the services, networks, and volumes required for your application in a declarative YAML file.

### Guide for using docker-compose for projects with multiple services
Link : https://www.honeybadger.io/blog/docker-django-react/

### Building and running multiple containers from Dockerfile
```
docker-compose up
```
### Sample Docker Compose file

```
services:
  frontend:                        #Define services here
  build: ./aws_tutorial            #Specify dockerfile location
    ports:
     - "3000:3000"
    volumes:                       #Maps host volume to container volume
     - ./aws_tutorial:/frontend
  backend:
    build: ./Backend
    environment:
      - SECRET_VARAIABLE=SECERET_VALUE
    ports:
      - "8000:8000"
    volumes:
      - ./Backend:/backend

```

Specifics on volumes : Mapping of host and container volume allows changes to sync between files in host machine and container. EX : if new code is added onto host machine, new code files will be added into container as well. EX: If volume is specified as /frontend:/frontendContainer and frontendContainer is not set as WORKDIR in Dockerfile, a new directory named frontendContainer will be created in the container.

### Tips when building CICD pipeline via Jenkins
Hide console outputs from docker-compose up by doing 

```
docker-compose up -d
```

# Docker container with MySQL
### Sample Dokcerfile hosting mysql DB

```
# Use an official MySQL 8.0 image as the base image
FROM mysql:8.0

# Set environment variables for MySQL configuration
ENV MYSQL_ROOT_PASSWORD=password


# Copy the SQL script to initialize the database
COPY ./init.sql /docker-entrypoint-initdb.d/

# Expose the MySQL port
EXPOSE 3306
```

### Sample Init.sql script for mysql
Used for creating a schema , filling in data for table
```
-- Create database
CREATE DATABASE mydatabase;

-- Use the database
USE mydatabase;

-- Create table
CREATE TABLE users (
  id INT AUTO_INCREMENT,
  name VARCHAR(50) NOT NULL,
  email VARCHAR(100) NOT NULL,
  PRIMARY KEY (id)
);

-- Insert sample data
INSERT INTO users (name, email) VALUES
('John Doe', 'john@example.com'),
('Jane Smith', 'jane@example.com');

```

### Using MySQL Workbench to connect to containerized DB
1. Allow inbound port for EC2 for port 3306
2. Build dockerfile and run the container with port mapping 3306:3306
3. In MySQL Workbench, open up new connection with TCP/SSH method
4. Procide public IP as hostname and ubuntu as ssh username
5. Provide .pem file for EC2 server authentication
6. Fill out MySQL credentials and test connection

### MySQL persistance

#### Scenario 1 : Stopping the container and starting the container again.
MySQL database data is persisted

### Scenario 2: Removing the container and building it from image again.
MySQL database data set to original state. MySQL Database in container in ephermeral on default.

### Scenario 3 : Removal of container after specifying volume mounts in docker-compose.
MySQL database data is persisted 

Sample docker-compose.yaml file for persistent db

```
services:
  database:
     build: ./Database
     ports:
      - "3306:3306"
     environment:
      - MYSQL_ROOT_PASSWORD=password
     volumes:
      -  mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```

In this example, the /var/lib/mysql data in the container is mounted on the host's mysql-data directory which is located in /var/lib/docker/volumes/mysql-data/_data. 





