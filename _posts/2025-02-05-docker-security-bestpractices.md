---
title: Docker Security Best Practices
date: 2025-02-05 11:20:05 +0530
categories: [Devsecops, Docker-Security]
tags: [Devsecops, Dockersecurity]
excerpt: Docker Security Best Practices
---


#  Docker Security Best Practices (Categorized by Lifecycle)

##  1. While Writing the Dockerfile

- **Use minimal and trusted base images**\
  *Why:* Lightweight images (like `alpine`, `distroless`) reduce the attack surface and minimize vulnerabilities, plus they make builds smaller and faster. Official images are vetted and maintained regularly.
- **Pin image versions** (avoid `latest`).\
  *Why:* Ensures consistency and prevents unexpected behavior from upstream image changes.
- **Run as non-root user**\
  *Why:* Containers running as root can be exploited to compromise the host. Running as a non-root user limits damage if compromised.
  ```dockerfile
  RUN addgroup -S appgroup && adduser -S appuser -G appgroup
  USER appuser
  ```
- **Reduce attack surface**\
  *Why:* Fewer tools and packages mean fewer potential vulnerabilities and exploits.
  - Remove unnecessary packages and debugging tools.
  - Use multi-stage builds to exclude build-time dependencies.
- **Do not store secrets in Dockerfile**\
  *Why:* Secrets baked into images can be extracted easily, leading to credential leaks.
- **Set explicit permissions** on files and directories.\
  *Why:* Prevents unauthorized access or modification inside the container.
- **Use .dockerignore** to exclude sensitive files.\
  *Why:* Prevents secrets and unnecessary files from being sent to the Docker daemon.

##  2. While Building the Image

- **Scan images during build** (Trivy, Grype, Snyk):\
  *Why:* Detect vulnerabilities early before pushing to production.
  
  ```bash
  trivy image myimage:tag
  ```
  
- **Immutable builds** (rebuild instead of patching running containers).\
  *Why:* Ensures a clear audit trail and avoids configuration drift.
- **Sign images** (Docker Content Trust or cosign):\
  *Why:* Guarantees image integrity and authenticity.
  ```bash
  export DOCKER_CONTENT_TRUST=1
  docker push myimage:tag
  ```
  
- **Use BuildKit** for secure builds:\
  *Why:* BuildKit isolates build secrets and offers better performance and caching.
  ```bash
  DOCKER_BUILDKIT=1 docker build .
  ```
- **Keep build cache clean** (`--no-cache` when needed).\
  *Why:* Prevents reusing potentially vulnerable layers.

##  3. While Deploying & Running Containers

- **Run containers as non-root**\
  *Why:* Limits the impact of a compromise.
  ```bash
  docker run --user 1000:1000 myimage:tag
  ```
- **Apply security profiles** (seccomp, AppArmor, SELinux):\
  *Why:* Restricts system calls and reduces the attack surface.
  ```bash
  docker run --security-opt seccomp=default.json myimage:tag
  ```
- **Limit capabilities**:\
  *Why:* Drops unneeded kernel privileges to mitigate exploitation.
  ```bash
  docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myimage:tag
  ```
- **Use read-only filesystems**:\
  *Why:* Prevents tampering with the filesystem during runtime.
  ```bash
  docker run --read-only --tmpfs /tmp myimage:tag
  ```
- **Restrict resource usage**:\
  *Why:* Prevents denial-of-service attacks by resource exhaustion.
  ```bash
  docker run --memory="256m" --cpus="0.5" --pids-limit=100 myimage:tag
  ```
- **Isolate networks** with custom bridge networks.\
  *Why:* Prevents lateral movement between containers.
- **Avoid privileged mode**.\
  *Why:* Privileged mode gives containers host-level access, which is dangerous.
- **Rotate secrets at runtime** (Docker Secrets or external managers).\
  *Why:* Reduces risk from stale or leaked credentials.
- **Use healthchecks** in Dockerfile or Compose.\
  *Why:* Detects unhealthy containers early and triggers self-healing mechanisms.

##  4. While Maintaining & Monitoring

- **Keep Docker Engine & Host OS updated**.\
  *Why:* Patches security vulnerabilities in both layers.
- **Continuous vulnerability scanning** (Harbor, AWS ECR, GitHub Security).\
  *Why:* Catches new CVEs affecting existing images.
- **Enable logging & monitoring** (Falco, Sysdig, Aqua Security).\
  *Why:* Helps detect suspicious runtime behavior.
- **Remove unused images and containers**:\
  *Why:* Reduces attack surface and disk usage.
  ```bash
  docker image prune -a
  docker container prune
  ```
- **Backup and disaster recovery** for image registries.\
  *Why:* Ensures quick recovery after a breach or data loss.
- **Implement RBAC** in CI/CD to restrict build/deploy permissions.\
  *Why:* Prevents unauthorized modifications or deployments.

##  Extra Best Practices (Often Missed)

- Use multi-stage builds to separate dev & prod dependencies.\
  *Why:* Reduces final image size and removes sensitive build-time data.
- Perform dependency scanning for code inside containers.\
  *Why:* Identifies vulnerabilities in application libraries.
- Verify image digests (`docker pull myimage@sha256:...`).\
  *Why:* Ensures you are pulling the exact, trusted image.
- Run containers in rootless mode where possible.\
  *Why:* Adds another layer of isolation from the host.
- Audit Docker daemon configuration (`/etc/docker/daemon.json`).\
  *Why:* Prevents insecure defaults like unauthenticated API access.
- Use private registries with authentication.\
  *Why:* Prevents unauthorized access or image tampering.

---


