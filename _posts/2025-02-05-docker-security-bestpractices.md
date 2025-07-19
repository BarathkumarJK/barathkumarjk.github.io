---
title: Docker Security Best Practices
date: 2025-02-05 11:20:05 +0530
categories: [Devsecops, Docker-Security]
tags: [Devsecops, Dockersecurity]
excerpt: Docker Security Best Practices
---


#  Docker Security Best Practices (Categorized by Lifecycle)

##  1. While Writing the Dockerfile

- **Use minimal and trusted base images**
  - Prefer lightweight images like `alpine` or `distroless`.
  - Use official or internally verified images only.
- **Pin image versions** (avoid `latest`).
- **Run as non-root user**
  ```dockerfile
  RUN addgroup -S appgroup && adduser -S appuser -G appgroup
  USER appuser
  ```
- **Reduce attack surface**
  - Remove unnecessary packages and debugging tools.
  - Use multi-stage builds to exclude build-time dependencies.
- **Do not store secrets in Dockerfile**
- **Set explicit permissions** on files and directories.
- **Use .dockerignore** to exclude sensitive files.

##  2. While Building the Image

- **Scan images during build** (Trivy, Grype, Snyk):
  ```bash
  trivy image myimage:tag
  ```
- **Immutable builds** (rebuild instead of patching running containers).
- **Sign images** (Docker Content Trust or cosign):
  ```bash
  export DOCKER_CONTENT_TRUST=1
  docker push myimage:tag
  ```
- **Use BuildKit** for secure builds:
  ```bash
  DOCKER_BUILDKIT=1 docker build .
  ```
- **Keep build cache clean** (`--no-cache` when needed).

##  3. While Deploying & Running Containers

- **Run containers as non-root**
  ```bash
  docker run --user 1000:1000 myimage:tag
  ```
- **Apply security profiles** (seccomp, AppArmor, SELinux):
  ```bash
  docker run --security-opt seccomp=default.json myimage:tag
  ```
- **Limit capabilities**:
  ```bash
  docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myimage:tag
  ```
- **Use read-only filesystems**:
  ```bash
  docker run --read-only --tmpfs /tmp myimage:tag
  ```
- **Restrict resource usage**:
  ```bash
  docker run --memory="256m" --cpus="0.5" --pids-limit=100 myimage:tag
  ```
- **Isolate networks** with custom bridge networks.
- **Avoid privileged mode**.
- **Rotate secrets at runtime** (Docker Secrets or external managers).
- **Use healthchecks** in Dockerfile or Compose.

##  4. While Maintaining & Monitoring

- **Keep Docker Engine & Host OS updated**.
- **Continuous vulnerability scanning** (Harbor, AWS ECR, Azure AKS, GitHub Security).
- **Enable logging & monitoring** (Falco, Sysdig, Aqua Security).
- **Remove unused images and containers**:
  ```bash
  docker image prune -a
  docker container prune
  ```
- **Backup and disaster recovery** for image registries.
- **Implement RBAC** in CI/CD to restrict build/deploy permissions.

##  Extra Best Practices (Often Missed)

- Use multi-stage builds to separate dev & prod dependencies.
- Perform dependency scanning for code inside containers.
- Verify image digests (`docker pull myimage@sha256:...`).
- Run containers in rootless mode where possible.
- Audit Docker daemon configuration (`/etc/docker/daemon.json`).
- Use private registries with authentication.

