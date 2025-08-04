# Docker Commands for Ubuntu Server Management

This document provides a comprehensive list of commands for managing Docker on an Ubuntu server, including installation, container and image management, and file cleanup tasks (e.g., using `sudo rm`). Each command includes its purpose and notes for safe usage. This document complements the Nginx commands guide for server administrators managing web services with Docker.

## Prerequisites
- **Ubuntu Server**: Ensure you have administrative access (sudo privileges).
- **Docker Installation**: Commands assume Docker is installed (see installation section below).
- **Access**: Commands are executed via SSH or terminal on the Ubuntu server.

## 1. Installing and Updating Docker
```bash
sudo apt update
sudo apt install -y docker.io
```
- **Purpose**: Updates package lists and installs Docker.
- **Note**: `docker.io` is the Docker package in Ubuntu's default repository. Use `-y` to auto-confirm installation.

```bash
sudo systemctl start docker
sudo systemctl enable docker
```
- **Purpose**: Starts Docker service and enables it to run on boot.
- **Note**: Verify Docker is running with `sudo systemctl status docker`.

```bash
sudo apt upgrade -y docker.io
```
- **Purpose**: Upgrades Docker to the latest version.
- **Note**: Stop Docker before upgrading to avoid conflicts:
  ```bash
  sudo systemctl stop docker
  ```

```bash
sudo usermod -aG docker $USER
```
- **Purpose**: Adds the current user to the Docker group to run Docker commands without `sudo`.
- **Note**: Log out and back in for the group change to take effect. Use cautiously to avoid security risks.

## 2. Managing Docker Containers
```bash
docker ps
```
- **Purpose**: Lists running containers.
- **Note**: Use `docker ps -a` to list all containers, including stopped ones.

```bash
docker run -d --name my_container -p 8080:80 nginx
```
- **Purpose**: Runs a container named `my_container` from the `nginx` image in detached mode, mapping port 8080 (host) to 80 (container).
- **Note**: Replace `nginx` with your desired image. Use `-d` for background running.

```bash
docker stop my_container
```
- **Purpose**: Stops the container named `my_container`.
- **Note**: Use `docker ps` to find container names or IDs.

```bash
docker start my_container
```
- **Purpose**: Starts a previously stopped container.
- **Note**: Retains previous configuration (e.g., port mappings).

```bash
docker restart my_container
```
- **Purpose**: Restarts a running container.
- **Note**: Useful for applying changes or recovering from issues.

```bash
docker rm my_container
```
- **Purpose**: Deletes a stopped container.
- **Note**: Stop the container first with `docker stop my_container`. Use `docker rm -f my_container` to force-remove a running container.

## 3. Managing Docker Images
```bash
docker pull nginx
```
- **Purpose**: Downloads the `nginx` image from Docker Hub.
- **Note**: Replace `nginx` with the desired image name and tag (e.g., `nginx:latest`).

```bash
docker images
```
- **Purpose**: Lists all Docker images on the system.
- **Note**: Displays image names, tags, IDs, and sizes.

```bash
docker rmi nginx
```
- **Purpose**: Deletes the `nginx` image.
- **Note**: Ensure no containers are using the image (remove them with `docker rm`). Use `docker rmi -f nginx` to force deletion.

```bash
sudo rm -rf /var/lib/docker/images/*
```
- **Purpose**: Deletes all Docker images from the storage directory.
- **Note**: Use with extreme caution. Stop Docker first (`sudo systemctl stop docker`) and ensure no important images are needed. Back up critical data before deletion.

## 4. Building Docker Images
```bash
docker build -t my_image .
```
- **Purpose**: Builds a Docker image named `my_image` from a `Dockerfile` in the current directory.
- **Note**: Ensure a `Dockerfile` exists. Example `Dockerfile` for Nginx:
  ```dockerfile
  FROM nginx:latest
  COPY index.html /usr/share/nginx/html/
  ```

## 5. Managing Docker Volumes
```bash
docker volume create my_volume
```
- **Purpose**: Creates a Docker volume named `my_volume`.
- **Note**: Volumes persist data outside containers.

```bash
docker run -d --name my_container -v my_volume:/app nginx
```
- **Purpose**: Runs a container with `my_volume` mounted at `/app` in the container.
- **Note**: Useful for persistent storage (e.g., for web content).

```bash
docker volume rm my_volume
```
- **Purpose**: Deletes a volume.
- **Note**: Ensure no containers are using the volume. Use `docker volume ls` to list volumes.

```bash
sudo rm -rf /var/lib/docker/volumes/my_volume
```
- **Purpose**: Manually deletes a volume's data from the filesystem.
- **Note**: Use with caution. Stop Docker (`sudo systemctl stop docker`) and verify the volume is unused.

## 6. Managing Docker Networks
```bash
docker network create my_network
```
- **Purpose**: Creates a custom Docker network named `my_network`.
- **Note**: Use for container-to-container communication.

```bash
docker run -d --name my_container --network my_network nginx
```
- **Purpose**: Runs a container connected to `my_network`.
- **Note**: Containers on the same network can communicate via their names.

```bash
docker network rm my_network
```
- **Purpose**: Deletes a network.
- **Note**: Ensure no containers are using the network (`docker network inspect my_network`).

## 7. Viewing Logs and Debugging
```bash
docker logs my_container
```
- **Purpose**: Displays logs for `my_container`.
- **Note**: Useful for debugging container issues.

```bash
docker exec -it my_container bash
```
- **Purpose**: Opens an interactive Bash shell inside `my_container`.
- **Note**: Replace `bash` with `sh` if Bash is unavailable. Use for inspecting or modifying container contents.

```bash
docker inspect my_container
```
- **Purpose**: Displays detailed configuration and status of `my_container`.
- **Note**: Useful for troubleshooting network or volume issues.

## 8. Cleaning Up Docker Resources
```bash
docker system prune
```
- **Purpose**: Removes stopped containers, unused networks, and dangling images.
- **Note**: Use `docker system prune -a` to remove all unused images (be cautious).

```bash
sudo rm -rf /var/lib/docker/containers/*
```
- **Purpose**: Deletes all container data from the Docker storage directory.
- **Note**: Extremely destructive. Stop Docker first (`sudo systemctl stop docker`) and back up critical containers or volumes.

## 9. Managing Docker Compose
```bash
sudo apt install -y docker-compose
```
- **Purpose**: Installs Docker Compose for managing multi-container applications.
- **Note**: Use for defining and running complex Docker setups via a `docker-compose.yml` file.

```bash
docker-compose up -d
```
- **Purpose**: Starts services defined in `docker-compose.yml` in detached mode.
- **Note**: Ensure a `docker-compose.yml` exists in the current directory. Example:
  ```yaml
  version: '3'
  services:
    web:
      image: nginx:latest
      ports:
        - "8080:80"
  ```

```bash
docker-compose down
```
- **Purpose**: Stops and removes containers defined in `docker-compose.yml`.
- **Note**: Add `--volumes` to remove associated volumes.

## 10. Firewall Configuration for Docker
```bash
sudo apt install -y ufw
sudo ufw allow 8080/tcp
```
- **Purpose**: Installs UFW and allows traffic on port 8080 (used in example `docker run` commands).
- **Note**: Adjust ports based on your application. Ensure SSH is allowed:
  ```bash
  sudo ufw allow OpenSSH
  sudo ufw enable
  ```

## Troubleshooting Tips
- **Permission Issues**: If `docker` commands fail without `sudo`, ensure you're in the `docker` group (`sudo usermod -aG docker $USER`).
- **Disk Space**: Monitor Docker storage with `docker system df`. Clean up unused resources with `docker system prune`.
- **Logs**: Always check container logs (`docker logs`) and Docker service status (`sudo systemctl status docker`) for errors.
- **Safety with `sudo rm`**: Verify paths before using `sudo rm -rf` (e.g., with `ls /var/lib/docker/`). Back up critical data:
  ```bash
  sudo cp -r /var/lib/docker /var/lib/docker_backup
  ```

## Additional Notes
- **Docker Storage**: Docker stores data in `/var/lib/docker/`. Be cautious when using `sudo rm -rf` in this directory, as it can delete critical images, containers, or volumes.
- **Nginx Integration**: To run Nginx in Docker, use the official `nginx` image (e.g., `docker run -d -p 80:80 nginx`). Map configuration files or web content using volumes:
  ```bash
  docker run -d -p 80:80 -v /path/to/nginx.conf:/etc/nginx/nginx.conf -v /path/to/html:/usr/share/nginx/html nginx
  ```
- **Documentation**: Keep this file updated with custom Docker commands or workflows specific to your setup.

For further assistance, consult the [Docker documentation](https://docs.docker.com/) or contact your server administrator.