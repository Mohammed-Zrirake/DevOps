#DevOps #Docker #CoreConcept #ImageBuilding #Tutorial

>  A `Dockerfile` is a text-based script containing a series of instructions that Docker follows to automatically build a [[Docker Image]]. It is the standard, repeatable, and version-controllable way to create your own images.

---

## 📜 Common Dockerfile Instructions

A `Dockerfile` is read from top to bottom. Here are some of the most common instructions:

-   `FROM <image>`: Specifies the base image to build upon. **Every `Dockerfile` must start with `FROM`**.
-   `WORKDIR <path>`: Sets the working directory for any subsequent `RUN`, `CMD`, `COPY`, etc., instructions inside the image.
-   `COPY <host-path> <image-path>`: Copies files or directories from your local machine (the build context) into the image's filesystem.
-   `RUN <command>`: Executes a command in a new layer on top of the current image. Used for installing packages, compiling code, etc.
-   `ENV <name>=<value>`: Sets a persistent environment variable inside the image.
-   `EXPOSE <port-number>`: Informs Docker that the container listens on the specified network port at runtime. This is primarily documentation; it doesn't actually publish the port.
-   `USER <user-or-uid>`: Sets the user name (or UID) to use when running the image and for any subsequent `RUN`, `CMD` instructions.
-   `CMD ["<executable>", "<param1>"]`: Provides the default command and arguments for an executing container. **There can only be one `CMD` instruction in a Dockerfile.**

> [!info] For a complete list of all instructions, see the official [Dockerfile reference](https://docs.docker.com/engine/reference/builder/).

---

## 🛠️ Hands-On: Building a Node.js Image

This guide will walk you through writing a `Dockerfile` to build a simple Node.js to-do list application.

### 1. Goal & Setup
-   **Goal:** To build a simple Node.js application image from scratch using a `Dockerfile`.
-   **Setup:**
    1.  Download and install Docker Desktop.
    2.  Download the project source code as a **[ZIP file](https://github.com/docker/getting-started-todo-app/archive/refs/heads/build-image-from-scratch.zip)** and extract it.
    3.  Navigate into the `getting-started-todo-app/app/` directory.

### 2. Create the Dockerfile
Inside the `app/` directory, you'll find an existing `Dockerfile`. For this exercise, **delete that file**.

Now, create a new, empty file named `Dockerfile` in the same directory.

> [!tip] No File Extension
> The file must be named `Dockerfile` exactly, with no file extension like `.txt`. Some editors may try to add one automatically.

### 3. Build the Image Step-by-Step

Add the following lines to your new `Dockerfile` one by one.

#### Step 3a: Define the Base Image (`FROM`)

This tells Docker to use the official Node.js version 22 image, specifically the lightweight `alpine` variant, as the starting point.

```dockerfile
FROM node:22-alpine
```

#### Step 3b: Set the Working Directory (`WORKDIR`)

This sets the working directory inside the image to `/app`. All subsequent commands will be run from this location. If the directory doesn't exist, Docker will create it.

```dockerfile
WORKDIR /app
```

#### Step 3c: Copy Project Files (`COPY`)

This copies all files from the current directory on your host machine (where the `Dockerfile` is) into the `/app` directory inside the image.

```dockerfile
COPY . .
```

#### Step 3d: Install Dependencies (`RUN`)

This executes the `yarn install` command inside the image to download and install the Node.js dependencies defined in `package.json`. The `--production` flag ensures only production dependencies are installed.

```dockerfile
RUN yarn install --production
```

#### Step 3e: Define the Startup Command (`CMD`)

This specifies the default command that will be executed when a container is started from this image. It will run the application by executing `node ./src/index.js`.

```dockerfile
CMD ["node", "./src/index.js"]
```

### 4. The Final Dockerfile

Your completed `Dockerfile` should look like this:

```dockerfile
# 1. Define the base image
FROM node:22-alpine

# 2. Set the working directory inside the container
WORKDIR /app

# 3. Copy all project files into the working directory
COPY . .

# 4. Install the application's dependencies
RUN yarn install --production

# 5. Specify the default command to run when the container starts
CMD ["node", "./src/index.js"]
```

### 5. Build and Run the Image

Now, open your terminal, make sure you are in the `app/` directory (where your `Dockerfile` is), and run the following command to build the image:

```bash
docker build -t getting-started .
```

-   `docker build`: The command to build an image.
-   `-t getting-started`: The `-t` flag "tags" the image with a name, in this case `getting-started`.
-   `.`: This tells Docker to use the current directory as the "build context" (where to find the `Dockerfile` and project files).

Once the build is complete, you can run your application in a container:

```bash
docker run -p 3000:3000 getting-started
```

---

> [!warning] This Dockerfile is Not Production-Ready
> This `Dockerfile` is intentionally simple for learning purposes. It doesn't follow all best practices for security or efficiency. Key improvements for a real-world scenario would include:
> -   **Optimizing the layer cache** (e.g., copying `package.json` first before copying all the source code).
> -   **Running the container as a non-root user** for better security.
> -   **Using a multi-stage build** to create a smaller, more secure final image.

> [!tip] The Modern Shortcut: `docker init`
> For new projects, the `docker init` command can analyze your project and automatically generate a best-practice `Dockerfile`, `.dockerignore`, and `docker-compose.yml` for you. It's a fantastic way to get started quickly once you understand the fundamentals.