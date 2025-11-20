#DevOps #Docker #Tutorial #HandsOn #ImageBuilding 

>  This is a hands-on tutorial demonstrating how to manually create [[Docker Image|Docker images]] by starting a [[Container]], making changes to its filesystem, and then "saving" those changes as a new image using the `docker container commit` command.

While the standard, repeatable way to build images is with a `[[Dockerfile]]`, this manual process is the best way to fundamentally understand how [[Docker Image Layers|image layers]] work.

---

## Part 1: Creating a Custom Base Image (`node-base`)

Our goal is to create a reusable base image that has Node.js pre-installed on top of a standard Ubuntu OS.

### 1. Start a Base Container
First, start an interactive container from the official `ubuntu` image. We'll name it `base-container` for easy reference.

```bash
docker run --name=base-container -ti ubuntu
```
*   `--name`: Assigns a specific name to the container.
*   `-ti`: A combination of `-t` (allocate a pseudo-TTY) and `-i` (interactive), which gives you a usable shell inside the container.

You will see a new shell prompt, which means you are now "inside" the container.
`root@d8c5ca119fcd:/#`

### 2. Install Node.js Inside the Container
Run the following commands to update the package manager and install Node.js. These changes are being made to the container's **writable layer**.

```bash
apt update && apt install -y nodejs
```

### 3. Verify the Installation
Check that Node.js was installed correctly.

```bash
node -e 'console.log("Hello world!")'
```
You should see `Hello world!` printed to the console.

### 4. Commit the Changes to a New Image
Now for the key step. Open a **new, separate terminal window** (do not exit the container's shell) and run the `docker container commit` command. This command takes a "snapshot" of the container's current state and saves it as a new image.

```bash
docker container commit -m "Add node" base-container node-base
```
*   `-m "Add node"`: Adds a commit message, which is useful for tracking history.
*   `base-container`: The name of the running container you want to save.
*   `node-base`: The name for your new image.

### 5. Inspect the Image History
You can now see the new layer you just created by inspecting the image's history.

```bash
docker image history node-base
```
The output will show your "Add node" layer sitting on top of the original Ubuntu base layers.
```
IMAGE          CREATED          CREATED BY     SIZE      COMMENT
d5c1fca2cdc4   10 seconds ago   /bin/bash      126MB     Add node
2b7cc08dcdbb   5 weeks ago      /bin/sh ...    69.2MB
...
```

### 6. Test the New Image and Clean Up
Prove that your new image works by running a new container from it. Then, remove the original container.

```bash
# Test the new image
docker run node-base node -e "console.log('Hello again')"

# Clean up the original container
docker rm -f base-container
```

> [!info] What is a Base Image?
> A **base image** is a foundation for building other images. In this case, `node-base` is a perfect base image because it provides a common starting point (an OS with Node.js installed) for any future Node.js applications you want to containerize.

---

## Part 2: Building an Application Image (`sample-app`)

Now, let's use our `node-base` image to build a final image that contains and runs a specific application.

### 1. Start a Container from Your New Base Image

```bash
docker run --name=app-container -ti node-base
```

### 2. Create the Application File
Inside this new container's shell, create a simple Node.js application file.

```bash
echo 'console.log("Hello from an app")' > app.js
```

### 3. Test the App Inside the Container

```bash
node app.js
```
You should see `Hello from an app` printed.

### 4. Commit to a Final Application Image
In a **separate terminal**, commit this container's changes to a new image named `sample-app`. This time, we'll use the `-c` flag to embed a default command into the image's metadata.

```bash
docker container commit -c "CMD node app.js" -m "Add app" app-container sample-app
```
*   `-c "CMD node app.js"`: Adds a Docker instruction to the new image's metadata. `CMD` specifies the default command to run when a container starts from this image.

### 5. Inspect the Final Image History
Check the history again. You will now see both of your custom layers stacked on top of each other.

```bash
docker image history sample-app
```
```
IMAGE          CREATED             CREATED BY     SIZE      COMMENT
c1502e2ec875   About a minute ago  /bin/bash      33B       Add app
5310da79c50a   4 minutes ago       /bin/bash      126MB     Add node
2b7cc08dcdbb   5 weeks ago         /bin/sh ...    69.2MB
...
```

### 6. Run the Final Image
Because you set a default `CMD`, you can now run the application without specifying a command.

```bash
# Run the final image
docker run sample-app

# Clean up the app container
docker rm -f app-container
```
You should see your "Hello from an app" greeting appear in the terminal immediately.

---

> [!best-practice] The Manual vs. The Automated Way
> Using `docker commit` is a powerful tool for debugging and for understanding how layers work. However, for real-world application development, it is **not the recommended practice**.
>
> The standard method is to define all these steps in a [[Dockerfile]]. A Dockerfile is a text-based script that provides a repeatable, version-controlled, and automated way to build your images, which is essential for any [[CI/CD]] pipeline.