#DevOps #Docker #BestPractice #ImageBuilding #Security #Performance

>  The best Docker images are small, secure, fast to build, and easy to maintain. This is achieved by using [[#Use multi-stage builds|multi-stage builds]], choosing minimal [[#Choose the right base image|trusted base images]], optimizing the [[#Leverage build cache|build cache]], and following specific guidelines for each [[Dockerfile]] instruction.

---

## 🏛️ High-Level Architectural Practices

### 1. Use Multi-Stage Builds
This is one of the most impactful best practices for production images.
- **Separate Build & Runtime:** [[Multi-Stage Docker Builds|Multi-stage builds]] use multiple `FROM` instructions to separate the build environment (with compilers, dev dependencies, etc.) from the final runtime environment.
- **Benefits:** This dramatically reduces the final image size and security attack surface by ensuring only the necessary application artifacts and a minimal runtime are included.

> [!tip] Create Reusable Stages
> If you have multiple Dockerfiles with common setup steps, create a shared base image or a reusable stage that includes those components. This follows the "Don't Repeat Yourself" (DRY) principle and makes builds faster and more efficient across your projects.

### 2. Choose the Right Base Image
The foundation of your image determines its size and security posture.
- **Use Trusted Sources:** Always prefer images from trusted sources to avoid vulnerabilities. Look for:
  - ✅ **[[Docker Official Images]]**: A curated, well-documented, and regularly updated set of images.
  - 🤝 **Verified Publishers**: High-quality images from commercial partners, verified by Docker.
  - 📦 **Docker-Sponsored Open Source**: Images maintained by sponsored open-source projects.
- **Keep it Minimal:** Choose the smallest base image that meets your needs (e.g., `alpine` variants or slimmed-down JREs instead of full JDKs). A smaller base image means faster downloads and a smaller attack surface.

### 3. Rebuild Your Images Often
Images are immutable snapshots. To incorporate security patches and updated dependencies from your base images and package managers, you must rebuild your images frequently.

To force a full refresh of all dependencies, you can bypass the build cache:
```bash
docker build --no-cache -t my-image:my-tag .
```

### 4. Create Ephemeral Containers
Design your images to produce containers that are as ephemeral (stateless) as possible. A container should be able to be stopped, destroyed, and replaced with a new one with minimal configuration. This aligns with the [Twelve-Factor App methodology](https://12factor.net/processes).

### 5. Decouple Applications
Follow the "one concern per container" principle. A web application stack should be composed of separate containers for the web server, the API, and the database. This makes them easier to scale, maintain, and reuse. Use [[Docker Networking]] to enable communication between them.

### 6. Pin Base Image Versions for Stability
A tag like `alpine:3.21` is mutable and can be updated by the publisher. This is good for getting security fixes but can introduce unexpected breaking changes.

- **For Predictability:** Pin the base image to a specific digest (`@sha256:...`). This guarantees you are always using the exact same image version for every build.
  ```dockerfile
  # This will ALWAYS use the same image, regardless of tag updates
  FROM alpine:3.21@sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c
  ```
- **The Trade-off:** Pinning means you opt-out of automatic security updates. Tools like Docker Scout can help by notifying you when a newer version of your pinned digest is available, allowing you to update deliberately.

---

## ⚡ Build Performance & Efficiency

### 1. Leverage the Build Cache
Docker reuses layers from previous builds to speed up the process. To maximize this, structure your `Dockerfile` from least-frequently-changed to most-frequently-changed instructions. See [[Optimizing Builds with the Docker Cache]] for a detailed guide.

### 2. Exclude Files with `.dockerignore`
To prevent irrelevant or sensitive files (like `.git`, `node_modules`, `.md` files) from being sent to the Docker daemon and invalidating the cache, create a `.dockerignore` file in your build context. Its syntax is similar to `.gitignore`.

### 3. Don't Install Unnecessary Packages
Avoid installing packages that aren't strictly required for the application to run (e.g., `vim`, `curl`, or build tools in a final runtime image). This reduces size, complexity, and potential vulnerabilities.

### 4. Sort Multi-line Arguments
For maintainability, sort multi-line arguments (like in a `RUN apt-get install`) alphanumerically. This makes the list easier to read, update, and review in pull requests.
```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends \
  bzr \
  cvs \
  git \
  mercurial \
  subversion \
  && rm -rf /var/lib/apt/lists/*
```

---

## 📜 Dockerfile Instruction Best Practices

### `FROM`
- Use current, minimal official images whenever possible (e.g., `alpine` variants).

### `LABEL`
- Use labels to add metadata to your image for organization, licensing, etc.
  ```dockerfile
  LABEL maintainer="hello@example.com"
  LABEL version="1.0"
  LABEL description="This is my awesome web application."
  ```

### `RUN`
- **Chain Commands:** Combine related commands with `&&` to reduce the number of layers.
- **Cache Busting for `apt-get`:** Always combine `apt-get update` with `apt-get install` in the same `RUN` statement. Also, clean up the cache at the end to reduce image size.
  ```dockerfile
  RUN apt-get update && apt-get install -y --no-install-recommends \
      package-foo \
      package-bar=1.3.* \
      && rm -rf /var/lib/apt/lists/*
  ```
- **Pipe Safety:** If you use pipes (`|`), prepend `set -o pipefail &&` to ensure the command fails if any part of the pipe fails.
  ```dockerfile
  RUN set -o pipefail && wget -O - https://some.site | wc -l > /number
  ```

### `CMD` and `ENTRYPOINT`
- **`ENTRYPOINT`:** Use it to set the image's main executable, making the image behave like that binary.
  ```dockerfile
  ENTRYPOINT ["s3cmd"]
  CMD ["--help"] # Provides default arguments
  ```
- **`CMD`:** Use it to provide default arguments for an `ENTRYPOINT` or to run an ad-hoc command (like an interactive shell) if no `ENTRYPOINT` is present. Always use the `exec` form: `CMD ["executable", "param1"]`.

### `COPY` vs. `ADD`
- **Prefer `COPY`:** It is more transparent and predictable. Use it for copying local files and directories from your build context.
- **Use `ADD` for Specific Features:** Use `ADD` only when you need its specific features, like fetching remote URLs or auto-extracting tar files.

### `ENV`
- Use `ENV` to set environment variables like `PATH` or application-specific configurations.
- To avoid leaving sensitive variables in layers, set, use, and unset them within a single `RUN` instruction.
  ```dockerfile
  RUN export MY_SECRET="super-secret" \
      && ./do-something-with-secret.sh \
      && unset MY_SECRET
  ```

### `EXPOSE`
- Use `EXPOSE` to document the port(s) the application listens on. It serves as information for the user and for tools.

### `VOLUME`
- Use `VOLUME` to designate directories that will store persistent data. This clearly indicates to users of the image which paths should be managed externally.

### `USER`
- **Run as Non-Root:** If a service can run without root privileges, always use `USER` to switch to a non-root user. Create the user and group first. This is a critical security best practice.
  ```dockerfile
  RUN groupadd -r myapp && useradd --no-log-init -r -g myapp myapp
  USER myapp
  ```

### `WORKDIR`
- Always use absolute paths for your `WORKDIR`. This makes your Dockerfile clearer and more reliable than chaining `cd` commands in `RUN` instructions.

### `ONBUILD`
- Use `ONBUILD` to create a trigger that will be executed in a "child" image that builds `FROM` your image. This is useful for creating generic base images for a specific language or framework that need to perform actions on the user's source code.