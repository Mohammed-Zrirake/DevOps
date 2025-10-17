#DevOps #Docker #CoreConcept #Networking #ResourceManagement #HandsOn

>  Use flags like `-p` (port), `-e` (environment), `--network`, and `--memory` with the `docker run` command to customize a [[Container|container's]] behavior on the fly, without needing to change the underlying [[Docker Image]].

---

## ❓ Why Override Defaults?

When a Docker container starts, it uses the default configuration defined in its image. While these defaults are often sensible, real-world scenarios require customization.

> [!info] Common Scenarios for Overriding Defaults
> - **Avoiding Port Conflicts:** Running multiple instances of the same service (e.g., two PostgreSQL databases for dev and test) on the same host requires mapping them to different host ports.
> - **Providing Configuration:** Applications often need specific configuration details (like database credentials or API keys) provided via environment variables.
> - **Managing Resources:** On a shared system, you need to limit a container's CPU and memory usage to prevent it from starving other processes.
> - **Network Isolation:** Connecting containers to specific custom networks for better security and service discovery.

The `docker run` command offers powerful flags to tailor a container's behavior to your exact needs.

---

## ⚙️ Common Runtime Overrides

### 1. Overriding Network Ports (`-p`)
The `-p` flag maps a port on the host to a port inside the container. This is covered in detail in [[Publishing Container Ports]].

```bash
# Runs a postgres container and maps host port 5433 to the container's default port 5432
docker run -d -p 5433:5432 postgres
```

### 2. Setting Environment Variables (`-e`, `--env-file`)
Use the `-e` flag to set an environment variable inside the container.

```bash
# Sets the environment variable 'foo' to the value 'bar' inside the container
docker run -e foo=bar postgres env
```
This would output all environment variables, including `foo=bar`.

> [!tip] Using a `.env` File
> To manage multiple environment variables cleanly, you can place them in a `.env` file and pass it to the container using the `--env-file` flag.
> ```bash
> docker run --env-file .env postgres env
> ```

### 3. Restricting Resources (`--memory`, `--cpus`)
You can limit a container's resource consumption to protect the host machine.

```bash
# Limits the container to 512MB of memory and half of a CPU core
docker run -d --memory="512m" --cpus="0.5" postgres
```

> [!tip] Monitoring Resource Usage
> Use the `docker stats` command in your terminal to see a live dashboard of the resource usage of all your running containers. This helps you determine if your limits are appropriate.

---

## 🛠️ Hands-On: Overriding Container Behavior

### 1. Run Multiple Postgres Instances
Start a standard Postgres container mapped to the default port `5432`.
```bash
docker run -d -e POSTGRES_PASSWORD=secret -p 5432:5432 postgres
```
Now, start a second instance. To avoid a port conflict, map it to a different host port, like `5433`.
```bash
docker run -d -e POSTGRES_PASSWORD=secret -p 5433:5432 postgres
```
You now have two independent Postgres databases running, accessible on your host at ports `5432` and `5433` respectively.

### 2. Use a Custom Network
By default, containers connect to a shared `bridge` network. For better isolation and DNS-based service discovery, it's best practice to use custom networks.

#### Step 2a: Create a Custom Network
```bash
docker network create mynetwork
```

#### Step 2b: Run a Container on the Custom Network
Use the `--network` flag to attach the container to your new network.
```bash
docker run -d -e POSTGRES_PASSWORD=secret -p 5434:5432 --network mynetwork postgres
```
This container is now isolated on `mynetwork`. Other containers on this same network can now discover and communicate with it using its container name as a hostname.

> [!info] Default Bridge vs. Custom Networks
> - **DNS Resolution:** On a custom network, containers can resolve each other by name (e.g., `ping postgres`). On the default bridge, they can only communicate by IP address (unless using legacy `--link`).
> - **Isolation:** A custom network provides a scoped environment. Unrelated containers on the default bridge cannot communicate with containers on your custom network, which is much more secure.

### 3. Override Default Commands (`CMD` / `ENTRYPOINT`)
You can override the image's default startup command directly from the command line or in a [[Docker Compose]] file.

#### With Docker Compose
Create a `compose.yaml` file. The `entrypoint` and `command` keys override the image defaults.
```yaml
services:
  postgres:
    image: postgres
    # Overrides the default ENTRYPOINT
    entrypoint: ["docker-entrypoint.sh"]
    # Overrides the default CMD, passing these as arguments to the entrypoint
    command: ["postgres", "-h", "localhost", "-p", "5432"]
    environment:
      POSTGRES_PASSWORD: secret
```
Start it with `docker compose up -d`.

#### With `docker run`
You can achieve the same result by specifying the new command *after* the image name.
```bash
docker run -d -e POSTGRES_PASSWORD=secret postgres docker-entrypoint.sh postgres -h localhost -p 5432
```
-   `postgres`: The image name.
-   `docker-entrypoint.sh postgres ...`: Everything after the image name becomes the new command that the container will execute, overriding the image's default `CMD` and `ENTRYPOINT`.