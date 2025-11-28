
#DevOps #CI-CD #Jenkins #CoreConcept #FreestyleJob #Configuration #BuildTriggers

>  This note provides a detailed, section-by-section breakdown of a typical `Freestyle Project` configuration page in [[Jenkins]]. It explains what each setting does and connects it to the practical goals of a Continuous Integration (CI) pipeline, such as managing disk space, triggering builds, running tests, and archiving results.

This interface represents the **Configuration Page of a Freestyle Job**.

---

## 1. General Section
This section defines the basic housekeeping and metadata for the job.

-   **`Discard old builds` (Log Rotation):**
    -   **Setting:** Checked.
    -   **`Max # of builds to keep`:** `10`.
    -   **Explanation:** Jenkins creates a record (logs, test results, [[#6 Post-build Actions|archived artifacts]]) for every single time this job runs. Without this setting, the Jenkins server's disk would fill up very quickly. This configuration tells Jenkins: **"Only keep the history of the last 10 runs. Automatically delete anything older."** This is a critical setting for managing disk space.

---

## 2. Source Code Management (SCM)
This section tells Jenkins where to find the application code it needs to build and test.

-   **`Git`:** The selected SCM provider. This tells Jenkins it will need to clone a Git repository.
-   **`Repository URL`:** `https://github.com/anshulc55/Jenkins_Upgradev3.git`
    -   **Explanation:** This is the specific remote address Jenkins will use to `git clone` or `git pull` the code.
-   **`Branch Specifier`:** `*/master`
    -   **Explanation:** Jenkins will only monitor and build code from the `master` branch of this repository. Commits pushed to other branches (e.g., `dev`, `feature-branch`) will be ignored by this specific job unless this setting is changed (e.g., to `*/main` or `*/dev`).

---

## 3. Build Triggers
This is a crucial section that defines **when** and **why** the job should run.

-   **`Build periodically`:**
    -   **Setting:** Checked.
    -   **Schedule:** `H/2 * * * *` (similar to instructor's `*/2 * * * *`). The `H` is a Jenkins-specific feature that means "hash" to spread load, so it might run at 1:03, 2:03, etc., instead of exactly on the hour.
    -   **Explanation:** This is a **time-based trigger**. It runs the job based on the clock, completely ignoring whether the code has changed or not. The syntax used is Cron.
    -   **Use Case (Nightly Builds):** This is ideal for running long, resource-intensive tasks (like full regression test suites or code analysis) overnight. For example, you could set it to run at 2 AM every day. When the team arrives in the morning, the results are ready.
    -   **Log Message:** When this trigger starts a build, the logs will show **"Started by timer"**.

-   **`Poll SCM`:**
    -   **Setting:** Checked.
    -   **Schedule:** `H/5 * * * *` (run every 5 minutes).
    -   **Explanation:** This is fundamentally different from `Build periodically`. Polling periodically asks the SCM (Git): **"Has any new code been committed since the last time I checked?"**
        -   If **YES**, the job runs.
        -   If **NO**, the job does nothing and waits for the next polling interval.
    -   **This is the core of Continuous Integration.** It ensures that every code change is built and tested automatically.

> [!tip] Combining Triggers
> You can use both `Build periodically` and `Poll SCM` on the same job. For example, `Poll SCM` every 5 minutes to catch immediate changes, and `Build periodically` every night at midnight to ensure a full, clean build and test run happens regardless of commit activity.
>
> **Modern Alternative:** For providers like GitHub, a more efficient method is to use **webhooks**, where the SCM server actively notifies Jenkins of a change, eliminating the need for constant polling.

---

## 4. Build Environment
These settings prepare the runner's environment *before* the main build steps are executed.

-   **`Delete workspace before build starts`:**
    -   **Explanation:** This option completely wipes out the job's workspace directory (the folder where Jenkins checked out the code) before every build.
    -   **Benefit:** This guarantees a **"clean" build**. It prevents issues where old files, temporary artifacts, or stale configurations from a previous run could interfere with the current one, making the build more reliable and reproducible.

-   **`Add timestamps to the Console Output`:**
    -   **Explanation:** This prepends the exact time (`HH:MM:SS`) to every line in the job's console log.
    -   **Benefit:** This is incredibly useful for debugging performance issues. You can easily see which specific step or command is taking the longest time to execute.

---

## 5. Build Steps
This is the heart of the job—the actual "work" it performs.

-   **`Execute shell`:**
    -   **Command:**
        ```bash
        cd $WORKSPACE
        cd maven-samples/single-module
        pwd
        ls
        mvn clean install
        ```
    -   **Explanation of Commands:**
        1.  `cd $WORKSPACE`: Moves the shell into the root directory of the job's workspace. `$WORKSPACE` is a Jenkins environment variable.
        2.  `cd maven-samples/single-module`: Navigates into the specific subfolder where the Java project's `pom.xml` file is located.
        3.  `pwd` and `ls`: Standard diagnostic commands to print the current directory and list its contents, useful for debugging.
        4.  `mvn clean install`: This is the **critical command**. It invokes **Maven**, a popular build tool for Java projects.
            -   `clean`: Deletes any previously compiled files and artifacts.
            -   `install`: Compiles the source code, runs all automated unit tests, and packages the application into a deployable artifact (in this case, a `.jar` file).

---

## 6. Post-build Actions
This section defines actions to take *after* the build steps have finished, regardless of success or failure.

-   **`Archive the artifacts`:**
    -   **`Files to archive`:** `maven-samples/single-module/target/*.jar`
    -   **Explanation:** The `mvn install` command creates a `.jar` file inside the `target/` directory. If you don't archive it, this file will be deleted the next time the workspace is cleaned. Archiving tells Jenkins to **save this final build artifact** and attach it to the build record. You can then download the built software directly from the Jenkins UI for that specific build number.

-   **`Publish JUnit test result report`:**
    -   **`Test report XMLs`:** `**/target/surefire-reports/*.xml`
    -   **Explanation:** While running tests, Maven generates standardized XML files that contain the detailed results of every test (pass, fail, skipped, duration). This post-build action tells Jenkins to find these XML files, parse them, and generate a **visual test result trend graph** on the job's dashboard. This provides a clear, historical view of test quality and stability (e.g., "10 tests passed, 2 failed").

-   **`Health report amplification factor`:**
    -   **Value:** `1.0`
    -   **Explanation:** This setting controls the sensitivity of the **"Weather" icon** (Sunny ☀️, Cloudy ☁️, Rainy 🌧️) on the Jenkins dashboard. The weather is a visual indicator of the job's recent health, often based on the percentage of passing tests. A factor of `1.0` means a direct correlation: if 80% of tests pass, the health is 80%. A higher factor would make the weather report "stormier" more quickly in response to failures.