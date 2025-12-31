# Docker Comprehensive Cheatsheet (Images → Containers → Docker Compose)

This cheatsheet covers **Docker end-to-end**, from fundamentals to real-world usage:

* Images and Dockerfiles
* Containers and lifecycle management
* Networking & volumes
* Docker Compose (multi-container apps)
* Best practices and production tips

All sections include **official documentation links** for deeper reference.

---

## 1. What Docker Is

Docker is a **containerization platform** that packages applications and their dependencies into **portable containers**.

Key ideas:

* *Image* → blueprint
* *Container* → running instance of an image

Docs: [https://docs.docker.com/get-started/overview/](https://docs.docker.com/get-started/overview/)

---

## 2. Core Docker Concepts (Mental Model)

```text
Dockerfile → Image → Container
                    ↳ Volumes
                    ↳ Networks
```

* Images are immutable
* Containers are ephemeral
* Data lives in volumes

---

## 3. Docker Installation & Setup

Install Docker Engine:

* Linux: [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/)
* macOS (Docker Desktop): [https://docs.docker.com/desktop/](https://docs.docker.com/desktop/)
* Windows: [https://docs.docker.com/desktop/windows/](https://docs.docker.com/desktop/windows/)

Verify:

```bash
docker version
docker info
```

---

## 4. Docker CLI Basics

### Check status

```bash
docker ps        # running containers
docker ps -a     # all containers
docker images    # local images
```

### Help

```bash
docker --help
docker run --help
```

CLI Docs: [https://docs.docker.com/engine/reference/commandline/docker/](https://docs.docker.com/engine/reference/commandline/docker/)

---

## 5. Docker Images

### Pull Image

```bash
docker pull nginx:latest
```

### List Images

```bash
docker images
```

### Remove Image

```bash
docker rmi nginx
```

Image Docs: [https://docs.docker.com/engine/reference/commandline/image/](https://docs.docker.com/engine/reference/commandline/image/)

---

## 6. Dockerfile (Building Images)

### Minimal Dockerfile

```Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

### Build Image

```bash
docker build -t myapp:1.0 .
```

Dockerfile Docs: [https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/)

---

## 7. Common Dockerfile Instructions

| Instruction | Purpose               |
| ----------- | --------------------- |
| FROM        | Base image            |
| RUN         | Execute command       |
| COPY / ADD  | Copy files            |
| WORKDIR     | Set working dir       |
| ENV         | Environment variables |
| EXPOSE      | Document port         |
| CMD         | Default command       |
| ENTRYPOINT  | Fixed command         |

---

## 8. Containers (Run & Manage)

### Run Container

```bash
docker run -d -p 8080:80 --name web nginx
```

### Stop / Start / Remove

```bash
docker stop web
docker start web
docker rm web
```

### Exec Into Container

```bash
docker exec -it web sh
```

Container Docs: [https://docs.docker.com/engine/reference/run/](https://docs.docker.com/engine/reference/run/)

---

## 9. Container Lifecycle

```text
Created → Running → Stopped → Removed
```

Restart policies:

```bash
docker run --restart unless-stopped nginx
```

---

## 10. Logs & Debugging

```bash
docker logs web
docker logs -f web
```

Inspect:

```bash
docker inspect web
```

---

## 11. Volumes (Persistent Data)

### Create Volume

```bash
docker volume create app-data
```

### Use Volume

```bash
docker run -v app-data:/data myapp
```

### Bind Mount

```bash
docker run -v $(pwd):/app myapp
```

Volume Docs: [https://docs.docker.com/storage/volumes/](https://docs.docker.com/storage/volumes/)

---

## 12. Networking

### Default Bridge Network

```bash
docker network ls
```

### Custom Network

```bash
docker network create mynet
docker run --network mynet nginx
```

Networking Docs: [https://docs.docker.com/network/](https://docs.docker.com/network/)

---

## 13. Environment Variables

```bash
docker run -e NODE_ENV=production myapp
```

From file:

```bash
docker run --env-file .env myapp
```

---

## 14. Docker Compose (Multi-Container Apps)

### What Compose Is

Docker Compose defines and runs **multi-container applications** using YAML.

Docs: [https://docs.docker.com/compose/](https://docs.docker.com/compose/)

---

## 15. docker-compose.yml (Basic Example)

```yaml
version: "3.9"
services:
  app:
    build: .
    ports:
      - "3000:3000"
    env_file:
      - .env
    depends_on:
      - db

  db:
    image: postgres:16
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

---

## 16. Docker Compose Commands

```bash
docker compose up
docker compose up -d
docker compose down
docker compose build
docker compose logs
```

---

## 17. Scaling Services

```bash
docker compose up --scale app=3
```

---

## 18. Healthchecks

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:3000"]
  interval: 30s
```

Docs: [https://docs.docker.com/engine/reference/builder/#healthcheck](https://docs.docker.com/engine/reference/builder/#healthcheck)

---

## 19. Docker Ignore (.dockerignore)

```text
node_modules
.git
.env
```

Docs: [https://docs.docker.com/engine/reference/builder/#dockerignore-file](https://docs.docker.com/engine/reference/builder/#dockerignore-file)

---

## 20. Tagging & Pushing Images (Docker Hub / ECR)

```bash
docker tag myapp:1.0 username/myapp:1.0
docker push username/myapp:1.0
```

Registry Docs: [https://docs.docker.com/docker-hub/](https://docs.docker.com/docker-hub/)

---

## 21. Docker + AWS (Context)

* Build locally or in GitHub Actions
* Push to ECR
* Run on EC2 / ECS / EKS

ECR Docs: [https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)

---

## 22. Best Practices

* Use small base images (alpine)
* Multi-stage builds
* One process per container
* Never bake secrets into images
* Use `.dockerignore`

Best Practices Docs: [https://docs.docker.com/develop/dev-best-practices/](https://docs.docker.com/develop/dev-best-practices/)

---

## 23. Typical Real-World Flow

```text
Dockerfile → Image → Container → Compose → CI/CD → AWS
```

---

## 24. Next Advanced Topics

* Multi-stage builds
* Docker security scanning
* ECS / Kubernetes manifests
* Blue/green deployments
* Observability (logs & metrics)

(Ask for **deep dives or production templates** anytime.)
