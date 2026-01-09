# Dockerfile

## Definition / Concept
A **Dockerfile** is a text file containing a series of instructions to build a **Docker image**. It's essentially a blueprint that automates the process of creating a containerized application by specifying the base image, dependencies, configuration, and commands needed to run an application. Docker images created from Dockerfiles provide consistent environments across development, testing, and production.

- Automates the image building process
- Ensures reproducible and consistent environments
- Makes applications portable across different systems
- Enables easy scaling and deployment

## Visual Representation
```
Dockerfile (Instructions)
    ↓
Docker Build Command
    ↓
Docker Image (Snapshot)
    ↓
Docker Run Command
    ↓
Docker Container (Running Instance)
```

## Example
```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

## Usage
- **When to use**: When you need to containerize applications for consistent deployment across environments
- **Real-world example**: Containerizing a Node.js backend service to run in Kubernetes clusters or Docker Swarm
- **Best practices**:
  - Use specific base image versions (avoid `latest`)
  - Minimize image size by using Alpine Linux
  - Combine `RUN` commands to reduce layers
  - Place frequently changing layers at the end
  - Use `.dockerignore` files to exclude unnecessary files

## FAQ / Interview Questions

**Q: What is the difference between a Docker image and a Docker container?**
A: A **Docker image** is a read-only template that contains everything needed to run an application (code, dependencies, environment). A **Docker container** is a running instance of that image. You can think of an image as a class and a container as an object instance.

**Q: What does the FROM instruction do?**
A: The **FROM** instruction specifies the **base image** from which your image is built. It's typically the first instruction and sets the foundation for all subsequent operations. For example, `FROM node:18-alpine` uses the official Node.js version 18 image based on Alpine Linux.

**Q: Why is layer caching important in Dockerfiles?**
A: Docker builds images in **layers**. Each instruction creates a new layer. Docker caches layers, so if you haven't changed a layer, it reuses it from cache, making subsequent builds faster. To maximize caching, place instructions that change frequently (like `COPY . .`) near the end.

**Q: What's the difference between RUN and CMD instructions?**
A: **RUN** executes commands during the image **build process** and commits the results to the image. **CMD** specifies the **default command to run** when a container starts. For example, `RUN npm install` runs during build, while `CMD ["node", "app.js"]` runs when the container starts.

**Q: How can you reduce Docker image size?**
A: Several techniques minimize image size:
- Use lightweight base images like Alpine Linux
- Install only necessary dependencies
- Combine `RUN` commands to reduce layers
- Remove package manager caches: `RUN npm ci && npm cache clean --force`
- Use **multi-stage builds** to exclude development dependencies from final image

## References
- [Docker Documentation - Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [Official Docker Image Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Docker - Best practices for writing Dockerfiles](https://docs.docker.com/develop/dev-best-practices/)

---
*See also: [Dockerfile Optimization](./DockerfileOptimization.md), [CI/CD](./CICD.md)*
