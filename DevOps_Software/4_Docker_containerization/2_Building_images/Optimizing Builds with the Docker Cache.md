
#DevOps #Docker #Performance #CoreConcept #ImageBuilding

>  Docker speeds up builds by reusing results (**layers**) from previous builds. To maximize this, structure your [[Dockerfile]] to place instructions that change infrequently (like installing dependencies) before instructions that change frequently (like copying your source code).

---

## ⚙️ How the Build Cache Works

When you run `docker build`, Docker executes each instruction in your [[Dockerfile]] sequentially, creating a new [[Docker Image Layers|layer]] for each command.

For each instruction, Docker first checks its internal cache to see if it has ever run this exact instruction before, starting from the same parent layer.

> [!success] Cache Hit
> If a valid cached layer exists, Docker doesn't re-execute the command. Instead, it reuses the existing layer, making the build significantly faster. You'll see `CACHED` in the build output.

### ⚠️ Cache Invalidation: The Key Concept

The build cache is powerful, but you need to understand what "invalidates" it. Once a layer's cache is invalidated, **all subsequent layers are also rebuilt from scratch**, regardless of whether they could have been cached.

Here are the most common causes of cache invalidation:

1.  **A `RUN` command changes:** Any modification to a `RUN` instruction invalidates that layer's cache.
2.  **Files from `COPY` or `ADD` change:** Docker calculates a checksum of the files being copied. If the checksum changes (meaning file content or metadata was modified), the cache for that `COPY` layer is invalidated.

> [!danger] The Invalidation Waterfall
> Once one layer is invalidated, all following layers in the `Dockerfile` are also invalidated and must be rebuilt. This is the most critical concept for optimization.

---

## 🛠️ Hands-On: Optimizing a Node.js Build

This tutorial will demonstrate a common caching mistake and how to fix it for dramatically faster builds.

### 1. Setup
1.  **Docker Desktop:** Ensure it's installed and running.
2.  **Clone the App:** Open a terminal and clone the sample application.
    ```bash
    git clone https://github.com/dockersamples/todo-list-app
    ```
3.  **Navigate:**
    ```bash
    cd todo-list-app
    ```

### 2. The Inefficient Build (Before Optimization)
Inside this directory, you'll find a `Dockerfile` with the following content:

```dockerfile
# Dockerfile: Inefficient Version
FROM node:22-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
EXPOSE 3000
CMD ["node", "./src/index.js"]
```
> **The Problem:** The `COPY . .` instruction is too early. Every time you change *any* file in your source code (e.g., a single line of HTML), the cache for the `COPY` layer is invalidated. Because of the waterfall effect, the subsequent `RUN yarn install` layer is *also* invalidated and must be re-run, even if your dependencies (`package.json`) haven't changed at all.

#### Step 2a: Run the First Build
Build the image for the first time. This will take a while as nothing is cached.
```bash
docker build .
```
The output will show a build time of around 20 seconds.

#### Step 2b: Run a Second Build (No Changes)
Run the build command again immediately, without changing any files.
```bash
docker build .
```
This build will be almost instant (around 1 second). The output will show `CACHED` for all the steps, proving the cache is working.

### 3. The Optimized Build (After Optimization)

Let's restructure the `Dockerfile` to be more cache-friendly. The strategy is to separate the dependency installation from the source code copying.

#### Step 3a: Update the Dockerfile
Update your `Dockerfile` to match the following. We copy only the files needed for dependency installation first, run `yarn install`, and *then* copy the rest of the source code.

```dockerfile
# Dockerfile: Optimized Version
FROM node:22-alpine
WORKDIR /app

# Copy only dependency manifests
COPY package.json yarn.lock ./

# Install dependencies (This layer is only invalidated if package.json or yarn.lock change)
RUN yarn install --production 

# Copy the rest of the application source code
COPY . . 

EXPOSE 3000
CMD ["node", "src/index.js"]
```

#### Step 3b: Create a `.dockerignore` File
To prevent your local `node_modules` folder from being copied into the image, create a file named `.dockerignore` in the same directory with the following content:
```
node_modules
```

#### Step 3c: Build the Optimized Image
Run the build command again. Since you changed the `Dockerfile`, it will rebuild all the layers.
```bash
docker build .
```

#### Step 3d: The Magic Moment - Test the Optimization
Now, make a small change to a source code file. For example, edit `src/static/index.html` and change the title to "The Awesome Todo App".

Build the image one more time.
```bash
docker build .
```
Look at the output carefully:
```
[+] Building 1.2s (10/10) FINISHED 
...
=> CACHED [2/5] WORKDIR /app                                                                      0.0s
=> CACHED [3/5] COPY package.json yarn.lock ./                                                    0.0s
=> CACHED [4/5] RUN yarn install --production                                                     0.0s
=> [5/5] COPY . .                                                                                 0.5s 
=> exporting to image                                                                             0.3s
...
```
**Success!** The build was incredibly fast. Docker was able to use the cache for the `yarn install` step because `package.json` didn't change. It only had to invalidate and re-run the final `COPY . .` step.

---

> [!summary] Key Takeaway for Optimization
> By following these optimization techniques, you can make your Docker builds faster and more efficient, leading to quicker iteration cycles and improved development productivity. The core principle is always: **order your Dockerfile instructions from least-frequently-changed to most-frequently-changed.**