#DevOps #Containerization #Kubernetes #CoreConcept #YAML #ConfigMap #ConfigurationAsCode #Manifests #HandsOn #Tutorial

>  A **`ConfigMap`** is a Kubernetes API object used to store **non-confidential configuration data** in a key-value pair format. It allows you to **decouple environment-specific configuration** from your container images, making your applications more portable and easier to manage.

---

## 🏛️ What is a ConfigMap?

The core concept behind a ConfigMap is to separate your application's configuration from its code. Instead of baking configuration files directly into your container image, you store them externally in a ConfigMap. This makes your container images generic and reusable across different environments (development, staging, production).

### How ConfigMaps Can Be Used
You can inject the data from a ConfigMap into a [[The Kubernetes Pod|Pod]] in several ways:
-   As **environment variables**.
-   As **command-line arguments**.
-   As **configuration files mounted into a volume**. (This is the method we will use in this demo).

---

## ✍️ Writing the `configmap.yaml` Manifest from Scratch

For our stateful application, we need our MySQL database to automatically create a default database (schema) when it first starts up. We can achieve this by providing a `.sql` initialization script. A `ConfigMap` is the perfect way to manage this script.

### The `configmap.yaml` Manifest
```yaml
# 1. API Version for ConfigMap is 'v1' (it's a core object)
apiVersion: v1
# 2. The kind of object is 'ConfigMap'
kind: ConfigMap
# 3. Metadata to identify the ConfigMap
metadata:
  name: user-management-db-creation-script
  
# 4. 'data' block contains the key-value configuration. There is no 'spec' for a ConfigMap.
data:
  # The 'key' is the filename we want to create: 'mysql-user-mgmt.sql'
  mysql-user-mgmt.sql: |-
    # The 'value' is a multi-line string containing the SQL script.
    -- Drop the database if it already exists to ensure a clean start
    DROP DATABASE IF EXISTS webappdb;
    
    -- Create the new database for our user management application
    CREATE DATABASE webappdb;
```

### A Deep Dive into the `ConfigMap` Structure
-   **`apiVersion` & `kind`:** `ConfigMap` is a core Kubernetes object, so its `apiVersion` is `v1`.
-   **`metadata.name`:** We give it a descriptive name that explains its purpose.
-   **`data`:** This is the key section for a ConfigMap. Unlike most other objects, it does **not** use a `spec` block.
    -   The `data` block is a dictionary of **key-value pairs**.
    -   **The Key:** In our case, the key (`mysql-user-mgmt.sql`) will become the **filename** when we mount this ConfigMap as a volume.
    -   **The Value:** The value is the content of that file. We use the YAML multi-line string literal (`|-`) to include our SQL script directly in the manifest.

### How it Will Be Used
In our upcoming MySQL Deployment manifest, we will:
1.  Create a `volume` that references this `user-management-db-creation-script` ConfigMap.
2.  Create a `volumeMount` inside the MySQL container that mounts this volume to a specific directory.
3.  The official MySQL container image is designed to automatically execute any `.sql` scripts it finds in its initialization directory (`/docker-entrypoint-initdb.d/`) upon its first startup.

By mounting our `mysql-user-mgmt.sql` file into that directory, we can declaratively ensure that our `webappdb` database is created automatically.

---

> [!summary] Conclusion
> You have successfully created a `ConfigMap` using a declarative YAML manifest. This is the standard way to manage non-sensitive configuration in Kubernetes. By externalizing our database initialization script into a ConfigMap, we keep our container image generic and our configuration explicit and version-controllable.