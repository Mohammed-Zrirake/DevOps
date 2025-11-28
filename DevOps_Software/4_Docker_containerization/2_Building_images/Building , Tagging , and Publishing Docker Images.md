#DevOps #Docker #Tutorial #HandsOn #ImageBuilding #Registry

>  This is the core workflow for image management.
> 1.  `docker build`: Creates a [[Docker Image]] from a [[Dockerfile]].
> 2.  `docker tag` (or `-t`): Gives the image a proper, shareable name.
> 3.  `docker push`: Uploads the tagged image to a [[Docker Registries & Repositories|registry]] like Docker Hub.

---

## 🏛️ The Three Core Concepts

### 1. Building Images (`docker build`)
This is the process of creating an image based on the instructions in a [[Dockerfile]].

The most basic `docker build` command tells Docker to use the `Dockerfile` in the current directory.
```bash
docker build .
```
-   The final `.` in the command specifies the **build context**. This is the set of files (and the `Dockerfile` itself) that are sent to the Docker daemon for the build.

When you run this, the builder pulls the base image (if needed) and then executes each instruction in your `Dockerfile`, creating a new layer for each step. The output will show the progress and, upon success, the ID of the newly created image.

```
$ docker build .
[+] Building 3.5s (11/11) FINISHED
...
=> => writing image sha256:9924dfd9350407b3df01d1a0e1033b1e543523ce7d5d5e2c83a724480ebe8f00
```
While you can run a container using this ID (`docker run 9924dfd9...`), it's not very memorable. That's where tagging comes in.

### 2. Tagging Images (`-t` and `docker tag`)
Tagging is the process of giving an image a memorable and structured name. A full image name has a specific format:
`[REGISTRY_HOST/][NAMESPACE/][REPOSITORY]:[TAG]`

-   **`REGISTRY_HOST`** (Optional): The hostname of the registry. Defaults to `docker.io` (Docker Hub).
-   **`NAMESPACE`** (Optional): Your username or organization on the registry. Defaults to `library` for official images.
-   **`REPOSITORY`**: The name of your application or image collection.
-   **`TAG`** (Optional): A version or variant identifier. Defaults to `latest`.

**Examples:**
-   `nginx` is equivalent to `docker.io/library/nginx:latest`
-   `docker/welcome-to-docker` is equivalent to `docker.io/docker/welcome-to-docker:latest`
-   `ghcr.io/dockersamples/example-voting-app-vote:pr-311` uses the GitHub Container Registry.

You can tag an image in two ways:
```bash
# 1. During the build (most common)
docker build -t your-username/my-image:v1 .

# 2. After the build, to an existing image
docker image tag <image-id> your-username/my-image:v2
```

### 3. Publishing Images (`docker push`)
Once an image is tagged correctly, you can push it to the registry specified in its name.

```bash
docker push your-username/my-image:v1
```

> [!warning] Authentication Required
> Before you can push an image to a private or user-specific repository, you must be authenticated. Use the `docker login` command to sign in to the registry (e.g., Docker Hub).
> ```bash
> docker login
> ```

---

## 🛠️ Hands-On: Build, Tag, and Push an Image

This tutorial will walk you through building a simple image and pushing it to your Docker Hub account.

### 1. Setup

1.  **Get the App:** Clone the sample application repository.

   ```bash
    git clone https://github.com/docker/getting-started-todo-app
    ```

2.  **Docker Desktop:** Ensure Docker Desktop is installed and running.

3.  **Docker Hub Account:** If you don't have one, [create a Docker Hub account](https://hub.docker.com/). Sign in to Docker Desktop with this account.

### 2. Build the Image

Navigate your terminal into the root of the cloned `getting-started-todo-app` repository. Run the following command, replacing `YOUR_DOCKER_USERNAME` with your actual Docker Hub username.

```bash
# Example: docker build -t mobywhale/concepts-build-image-demo .
docker build -t YOUR_DOCKER_USERNAME/concepts-build-image-demo .
```

-   This command builds the image using the `Dockerfile` in the current directory.
-   It tags (`-t`) the image with your username and a repository name, ready for pushing to Docker Hub.

### 3. Inspect the Image

Once the build is complete, you can see your new image in your local image list.

```bash
docker image ls
```

You'll see output similar to this, showing your newly created repository:

```
REPOSITORY                                TAG       IMAGE ID       CREATED          SIZE
your-username/concepts-build-image-demo   latest    746c7e06537f   24 seconds ago   354MB
```

You can also view the layers that make up your image:

```bash
docker image history YOUR_DOCKER_USERNAME/concepts-build-image-demo
```

### 4. Push the Image

Now, push the image to your Docker Hub repository.

```bash
docker push YOUR_DOCKER_USERNAME/concepts-build-image-demo
```

> [!tip] Access Denied?
> If you get a "requested access to the resource is denied" error, double-check two things:
> 1.  You are logged in (`docker login`).
> 2.  The username in your image tag exactly matches your Docker Hub username.

After a few moments, the layers of your image will be uploaded. You can now log in to Docker Hub in your browser and see your new repository!