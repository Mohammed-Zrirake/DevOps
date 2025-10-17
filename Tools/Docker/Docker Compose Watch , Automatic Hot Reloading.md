#DevOps #Docker #Compose #Development #Workflow

>  `Compose Watch` is a feature that automatically updates your running services when you save your local code. It's a powerful companion to [[Bind Mounts _ Linking Host Files to Containers]] that offers more granular control, enabling a true "hands-off" development workflow with hot reloading.

---

## 🚀 What is Compose Watch?

The `watch` attribute in a `compose.yaml` file tells Docker Compose to monitor your local files for changes and automatically take action—either **syncing** the changes into the container or **rebuilding** the service entirely.

For many projects, this enables a seamless development workflow. Once you run `docker compose up --watch`, your services update themselves in real-time as you save your work in your IDE.

### ⚠️ Prerequisites
For `watch` to function correctly, your service's image must contain a few common executables (`stat`, `mkdir`, `rmdir`) and the container's user must have write permissions to the target paths. To ensure correct ownership when copying files, use the `--chown` flag in your [[Dockerfile]].
```dockerfile
# Run as a non-privileged user for security
RUN useradd -ms /bin/sh -u 1001 app
USER app

WORKDIR /app
# Copy files and ensure they are owned by the 'app' user
COPY --chown=app:app . /app
```

---

## 🆚 Watch vs. Bind Mounts: A Modern Companion

`Compose Watch` does not replace [[Bind Mounts]] but acts as a more intelligent companion specifically for development.

> [!tip] The Key Advantage: Granularity
> The biggest advantage of `watch` is its ability to **ignore** specific files or directories. This is critical for performance and multi-platform development.

For example, in a Node.js project, ignoring the `node_modules/` directory has two major benefits:
1.  **Performance:** It prevents the high I/O load caused by syncing thousands of small dependency files.
2.  **Cross-Platform Compatibility:** It avoids syncing compiled native artifacts from your host (e.g., macOS) into a Linux container where they would be incompatible.

---

## ⚙️ Configuration (`develop.watch`)

The `watch` attribute is defined under a `develop` key for a service. It's a list of rules, each with a `path` to watch and an `action` to take.

### Core `watch` Actions

#### 1. `action: sync`
- **What it does:** When a file in `path` changes, Compose copies the updated file to the `target` path inside the container.
- **Ideal for:** Interpreted languages (JavaScript, Python, Ruby) and frameworks that support **Hot Reloading**. This is the modern replacement for a simple bind mount for source code.

#### 2. `action: rebuild`
- **What it does:** When a file in `path` changes, Compose rebuilds the entire image and recreates the service container. This is equivalent to running `docker compose up --build <service>`.
- **Ideal for:**
  - Changes to files that require a full rebuild (e.g., `package.json`, `pom.xml`, `Dockerfile`).
  - Compiled languages where a code change requires recompilation.

#### 3. `action: sync+restart`
- **What it does:** Syncs the file changes into the container (like `sync`) and then restarts the service.
- **Ideal for:** Changes to configuration files (e.g., `nginx.conf`, database configs) that require a process restart to take effect, but not a full image rebuild.

---

### Other Configuration Fields

- **`path`:** The local path to watch, relative to the project directory.
- **`target`:** The destination path inside the container.
- **`ignore`:** A list of patterns (relative to `path`) to exclude from watching. This is a key feature.
- **`initial_sync`** (boolean): For `sync` actions, tells Compose to perform an initial sync of all files when watch mode starts.

---

## 📝 Examples

### Example 1: Node.js Hot Reloading
This is a classic pattern for a frontend application.

**File Structure:**
```
myproject/
├── web/
│   ├── App.jsx
│   └── node_modules/
├── Dockerfile
├── compose.yaml
└── package.json
```

**`compose.yaml`:**
```yaml
services:
  web:
    build: .
    command: npm start # Assumes this runs a dev server with hot reload
    develop:
      watch:
        # Rule 1: For source code changes, just sync the file
        - action: sync
          path: ./web
          target: /src/web
          ignore:
            - node_modules/
        
        # Rule 2: For dependency changes, trigger a full rebuild
        - action: rebuild
          path: package.json
```
**Workflow:**
1.  Run `docker compose up --watch`.
2.  Change a file like `web/App.jsx`. `watch` syncs it to `/src/web/App.jsx` in the container. The running dev server (Vite, Webpack) detects the change and hot-reloads the app in your browser.
3.  Change `package.json`. `watch` triggers a full `docker build` and restarts the service with the new dependencies.

### Example 2: `sync+restart` for Configuration
This example shows a service that needs to be restarted when its configuration file changes.
```yaml
services:
  web:
    build: .
    develop:
      watch:
        - action: sync
          path: ./src
          target: /app/src
        
        # Rule: If nginx.conf changes, sync it and restart the service
        - action: sync+restart
          path: ./proxy/nginx.conf
          target: /etc/nginx/conf.d/default.conf
```

---

## ▶️ How to Use Watch

1.  Add a `develop.watch` section to one or more services in your `compose.yaml`.
2.  Run your project with the `--watch` flag:
    ```bash
    docker compose up --watch
    ```
3.  Start editing your code!

> [!tip] A Dedicated Watch Command
> If you prefer to keep your application logs separate from the watch/sync/build logs, you can run the services first and then start watch mode in a separate terminal:
> ```bash
_Terminal 1:_
> docker compose up
>
_Terminal 2:_
> docker compose watch
> ```