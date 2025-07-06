## 1. Docker commands to build and push

```bash
# Build the image
docker build -t crashrik/learner-report-backend:latest .
docker push crashrik/learner-report-backend:latest

docker build -t crashrik/learner-report-frontend:latest .
docker push crashrik/learner-report-frontend:latest
```
## 2. Quick fix - try building with more verbose output

Run this command to see the actual error:

```bash
docker build --no-cache --progress=plain -t your-frontend .
```


## 1. Stop and remove containers from docker-compose

```bash
# Stop and remove all containers defined in docker-compose.yml
docker-compose down

# Stop and remove containers + volumes + networks
docker-compose down -v

# Stop and remove containers + images
docker-compose down --rmi all
```

## 2. Delete specific containers

```bash
# Stop containers first
docker stop learner-report-backend learner-report-frontend

# Remove containers
docker rm learner-report-backend learner-report-frontend

# Or combine stop and remove
docker rm -f learner-report-backend learner-report-frontend
```

## 3. Delete all stopped containers

```bash
# Remove all stopped containers
docker container prune

# Remove all containers (running and stopped) - BE CAREFUL
docker rm -f $(docker ps -aq)
```

## 4. Complete cleanup commands

```bash
# Remove containers, networks, and images created by docker-compose
docker-compose down --rmi all --volumes --remove-orphans

# Clean up system (removes unused containers, networks, images)
docker system prune

# Aggressive cleanup (removes everything not in use)
docker system prune -a --volumes
```

## 5. Check current containers

```bash
# List running containers
docker ps

# List all containers (running and stopped)
docker ps -a

# List containers from your compose file
docker-compose ps
```

**For your specific setup**, run:
```bash
docker-compose down
```

This will stop and remove the `learner-report-backend` and `learner-report-frontend` containers along with the `learner-report-network`.