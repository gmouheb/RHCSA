# Containers in Linux

This document covers container concepts, tools, and management in Linux systems.

## Introduction to Containers

Containers are lightweight, standalone, executable packages that include everything needed to run an application: code, runtime, system tools, libraries, and settings. Containers provide:

- Application isolation
- Consistent environments across development, testing, and production
- Efficient resource utilization
- Rapid deployment and scaling
- Microservices architecture support

## Container vs. Virtual Machines

| Feature | Containers | Virtual Machines |
|---------|------------|------------------|
| Isolation | Process-level isolation | Hardware-level isolation |
| Size | Lightweight (MBs) | Heavyweight (GBs) |
| Boot time | Seconds | Minutes |
| Performance | Near-native | Overhead due to hypervisor |
| OS | Share host kernel | Complete OS per VM |
| Resource efficiency | High | Lower |

## Container Technologies

### Docker

Docker is the most popular container platform, providing tools to build, run, and manage containers.

### Podman

Podman is a daemonless container engine for developing, managing, and running OCI containers. It's the default container tool in RHEL/CentOS 8 and later.

### LXC/LXD

LXC (Linux Containers) provides OS-level virtualization through a virtual environment with its own process and network space.

### Container Orchestration

- **Kubernetes**: Open-source platform for automating deployment, scaling, and management of containerized applications
- **OpenShift**: Red Hat's enterprise Kubernetes platform
- **Docker Swarm**: Docker's native clustering and orchestration solution

## Installing Container Tools

### Installing Docker

```bash
# On RHEL/CentOS
dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
dnf install docker-ce docker-ce-cli containerd.io

# Start and enable Docker
systemctl enable --now docker

# Verify installation
docker --version
docker info
```

### Installing Podman

```bash
# On RHEL/CentOS
dnf install podman

# Verify installation
podman --version
podman info
```

## Basic Container Operations

### Docker Commands

```bash
# Pull an image
docker pull nginx

# List images
docker images

# Run a container
docker run -d --name web -p 8080:80 nginx

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Stop a container
docker stop web

# Start a container
docker start web

# Remove a container
docker rm web

# Remove an image
docker rmi nginx
```

### Podman Commands

Podman commands are largely compatible with Docker:

```bash
# Pull an image
podman pull nginx

# List images
podman images

# Run a container
podman run -d --name web -p 8080:80 nginx

# List running containers
podman ps

# List all containers (including stopped)
podman ps -a

# Stop a container
podman stop web

# Start a container
podman start web

# Remove a container
podman rm web

# Remove an image
podman rmi nginx
```

## Working with Container Images

### Finding Images

```bash
# Search for images
docker search ubuntu
podman search ubuntu

# View image details
docker inspect nginx
podman inspect nginx
```

### Building Custom Images with Dockerfile

A Dockerfile is a text file that contains instructions for building a container image:

```dockerfile
# Example Dockerfile
FROM centos:8
MAINTAINER Your Name <your.email@example.com>

# Install packages
RUN dnf -y update && dnf -y install httpd

# Copy application files
COPY ./app/ /var/www/html/

# Expose port
EXPOSE 80

# Set environment variables
ENV APACHE_LOG_DIR /var/log/httpd

# Define volume
VOLUME /var/www/html

# Start Apache
CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]
```

### Building and Tagging Images

```bash
# Build an image from a Dockerfile
docker build -t myapp:1.0 .
podman build -t myapp:1.0 .

# Tag an image
docker tag myapp:1.0 myregistry.example.com/myapp:1.0
podman tag myapp:1.0 myregistry.example.com/myapp:1.0
```

### Working with Registries

```bash
# Log in to a registry
docker login registry.example.com
podman login registry.example.com

# Push an image to a registry
docker push myregistry.example.com/myapp:1.0
podman push myregistry.example.com/myapp:1.0

# Pull an image from a registry
docker pull myregistry.example.com/myapp:1.0
podman pull myregistry.example.com/myapp:1.0
```

## Container Networking

### Default Networks

```bash
# List networks
docker network ls
podman network ls

# Inspect a network
docker network inspect bridge
podman network inspect podman
```

### Creating Custom Networks

```bash
# Create a bridge network
docker network create mynetwork
podman network create mynetwork

# Run a container on a specific network
docker run -d --network=mynetwork --name=container1 nginx
podman run -d --network=mynetwork --name=container1 nginx
```

### Network Port Mapping

```bash
# Map container port to host port
docker run -d -p 8080:80 nginx
podman run -d -p 8080:80 nginx

# Map multiple ports
docker run -d -p 8080:80 -p 8443:443 nginx
podman run -d -p 8080:80 -p 8443:443 nginx

# Map to specific host IP
docker run -d -p 127.0.0.1:8080:80 nginx
podman run -d -p 127.0.0.1:8080:80 nginx
```

## Container Storage

### Volumes

Volumes are the preferred way to persist data in containers:

```bash
# Create a volume
docker volume create mydata
podman volume create mydata

# List volumes
docker volume ls
podman volume ls

# Run a container with a volume
docker run -d -v mydata:/app/data nginx
podman run -d -v mydata:/app/data nginx

# Inspect a volume
docker volume inspect mydata
podman volume inspect mydata

# Remove a volume
docker volume rm mydata
podman volume rm mydata
```

### Bind Mounts

Bind mounts map a host directory to a container directory:

```bash
# Mount a host directory to a container
docker run -d -v /host/path:/container/path nginx
podman run -d -v /host/path:/container/path nginx

# Mount with read-only option
docker run -d -v /host/path:/container/path:ro nginx
podman run -d -v /host/path:/container/path:ro nginx
```

## Container Resource Management

### Setting Resource Limits

```bash
# Limit CPU
docker run -d --cpus=0.5 nginx
podman run -d --cpus=0.5 nginx

# Limit memory
docker run -d --memory=512m nginx
podman run -d --memory=512m nginx

# Set CPU priority
docker run -d --cpu-shares=512 nginx
podman run -d --cpu-shares=512 nginx
```

### Monitoring Container Resources

```bash
# View container stats
docker stats
podman stats

# View stats for a specific container
docker stats container_id
podman stats container_id
```

## Container Orchestration with Docker Compose

Docker Compose is a tool for defining and running multi-container applications:

### Installing Docker Compose

```bash
# Install Docker Compose
curl -L "https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

# Verify installation
docker-compose --version
```

### Example docker-compose.yml

```yaml
version: '3'
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    depends_on:
      - db
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: app
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

### Using Docker Compose

```bash
# Start services
docker-compose up -d

# View running services
docker-compose ps

# View logs
docker-compose logs

# Stop services
docker-compose down

# Stop services and remove volumes
docker-compose down -v
```

## Podman Pods

Podman supports pods, which are groups of containers that share resources:

```bash
# Create a pod
podman pod create --name mypod -p 8080:80

# Run containers in a pod
podman run -d --pod mypod --name web nginx
podman run -d --pod mypod --name db mysql:5.7

# List pods
podman pod list

# Stop a pod
podman pod stop mypod

# Remove a pod
podman pod rm mypod
```

## Container Security

### Running Containers with Reduced Privileges

```bash
# Run as non-root user
docker run -d --user 1000:1000 nginx
podman run -d --user 1000:1000 nginx

# Drop capabilities
docker run -d --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx
podman run -d --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx

# Run with read-only filesystem
docker run -d --read-only nginx
podman run -d --read-only nginx
```

### Security Scanning

```bash
# Scan images for vulnerabilities (Docker)
docker scan nginx

# Scan images with Trivy (works with both Docker and Podman)
trivy image nginx
```

### SELinux and Containers

```bash
# Run container with SELinux context
docker run -d --security-opt label=type:container_t nginx
podman run -d --security-opt label=type:container_t nginx

# View SELinux context of running containers
ps -eZ | grep container
```

## Rootless Containers with Podman

Podman supports running containers as a non-root user:

```bash
# Run as regular user
# No need for sudo with Podman
podman run -d --name web nginx

# Configure user namespace mapping
podman system migrate
```

## Systemd Integration

### Running Containers as Systemd Services

With Podman, you can generate systemd unit files for containers:

```bash
# Generate systemd unit file
podman generate systemd --name web > /etc/systemd/system/container-web.service

# Reload systemd
systemctl daemon-reload

# Enable and start the container service
systemctl enable --now container-web

# Check status
systemctl status container-web
```

## Container Troubleshooting

### Viewing Container Logs

```bash
# View container logs
docker logs container_id
podman logs container_id

# Follow log output
docker logs -f container_id
podman logs -f container_id

# Show timestamps
docker logs --timestamps container_id
podman logs --timestamps container_id
```

### Executing Commands in Running Containers

```bash
# Execute a command in a container
docker exec -it container_id /bin/bash
podman exec -it container_id /bin/bash

# Execute as a specific user
docker exec -it -u root container_id /bin/bash
podman exec -it -u root container_id /bin/bash
```

### Inspecting Container Details

```bash
# View detailed container information
docker inspect container_id
podman inspect container_id

# View specific information using format
docker inspect --format='{{.NetworkSettings.IPAddress}}' container_id
podman inspect --format='{{.NetworkSettings.IPAddress}}' container_id
```

### Common Issues and Solutions

1. **Container won't start**:
   ```bash
   # Check logs
   docker logs container_id
   
   # Check if ports are already in use
   ss -tulpn | grep 8080
   ```

2. **Network connectivity issues**:
   ```bash
   # Check container IP
   docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_id
   
   # Test network from inside container
   docker exec container_id ping 8.8.8.8
   ```

3. **Storage issues**:
   ```bash
   # Check available space
   df -h
   
   # Check volume mounts
   docker inspect -f '{{.Mounts}}' container_id
   ```

## Best Practices

1. **Use specific image tags** instead of `latest`
2. **Keep images small** by using multi-stage builds and minimal base images
3. **Don't run containers as root** when possible
4. **Use volumes for persistent data** instead of storing data in containers
5. **Implement health checks** to monitor container health
6. **Scan images for vulnerabilities** before deployment
7. **Use resource limits** to prevent container resource abuse
8. **Document container configurations** with docker-compose or similar tools
9. **Implement proper logging** for containers
10. **Regularly update base images** to get security patches