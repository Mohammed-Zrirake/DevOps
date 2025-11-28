#CI-CD #GitHub #Automation #Workflow #Scripting #API #Octokit #HandsOn

>  Go beyond simple comments by using [[GitHub Script]] to automate advanced tasks like managing project boards. Improve your workflows by separating logic into multiple [[#⚙️ Separating Logic into Steps|steps]], using `if` conditionals to control their execution, and leveraging the full [[#📂 Leveraging the Node.js Environment|Node.js environment]] to interact with files in your repository.

---

## 🤖 Automating Project Boards

In addition to creating comments, you can use the authenticated Octokit client provided by GitHub Script to manage [[GitHub Projects]]. A common use case is to automatically add newly created issues to a triage board.

**Example: A workflow that comments on a new issue AND adds it to a project board.**
```yaml
name: Triage New Issues
on:
  issues:
    types: [opened]
jobs:
  triage:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/github-script@v7
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}
        script: |
          // First, create the acknowledgement comment
          github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: "🎉 Thank you for opening this issue!"
          })

          // Next, add the issue as a card to a project board column
          github.rest.projects.createCard({
            column_id: 12345678, // Replace with your actual Column ID
            content_id: context.payload.issue.id,
            content_type: "Issue"
          });
```
> [!warning] Placeholder Value
> The `column_id` in the example above is a placeholder. You must replace `12345678` with the actual ID of the column from your GitHub project board. You can find this ID via the GitHub API or by inspecting the URL of the project board column.

---

## ⚙️ Separating Logic into Steps

A key benefit of GitHub Actions is the ability to break down complex jobs into smaller, independent `steps`. This has two major advantages:
1.  **Clarity:** It makes the workflow easier to read and debug, with each step having a clear name and purpose in the Actions UI.
2.  **Control:** It allows you to apply conditional logic (`if`) to run certain steps only when specific conditions are met.

Let's refactor the previous example into two distinct steps.

```yaml
name: Triage New Issues with Steps
on:
  issues:
    types: [opened]
jobs:
  triage:
    runs-on: ubuntu-latest
    steps:
    - name: Comment on new issue
      uses: actions/github-script@v7
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}
        script: |
          github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: "🎉 Thank you for opening this issue!"
          })

    - name: Add 'bug' issue to project board
      # This step will ONLY run if the issue has the "bug" label
      if: contains(github.event.issue.labels.*.name, 'bug')
      uses: actions/github-script@v7
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}
        script: |
          github.rest.projects.createCard({
            column_id: 12345678, // Replace with your actual Column ID
            content_id: context.payload.issue.id,
            content_type: "Issue"
          });
```
-   **`name:`:** Each step now has a descriptive name, which is shown in the workflow logs.
-   **`if:`:** The second step includes an `if` conditional. The expression `contains(github.event.issue.labels.*.name, 'bug')` checks if the array of labels on the triggering issue includes the string "bug". The step is completely skipped if this condition is false.

---

## 📂 Leveraging the Node.js Environment: Reading Files

GitHub Script runs in a full Node.js environment, which means you can use built-in Node modules to perform more complex tasks, like interacting with the filesystem.

> [!tip] Prerequisite: `actions/checkout`
> To read files from your repository, you must first check out the repository's code into the runner's workspace. This is done by adding the `actions/checkout` action as the first step in your job.

**Example: A workflow that reads a comment from a template file and posts it to new issues.**

Let's assume you have a file in your repository at `.github/ISSUE_RESPONSES/comment.md`.

**The final, fully-featured workflow:**
```yaml
name: Advanced Issue Triage
on:
  issues:
    types: [opened]
jobs:
  triage:
    runs-on: ubuntu-latest
    steps:
      # Step 1: Check out the repository's code
      - name: Checkout repo
        uses: actions/checkout@v4

      # Step 2: Read a file and use its content to create a comment
      - name: Comment on new issue from template
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
             const fs = require('fs');
             const issueBody = fs.readFileSync(".github/ISSUE_RESPONSES/comment.md", "utf8");
             
             github.rest.issues.createComment({
               issue_number: context.issue.number,
               owner: context.repo.owner,
               repo: context.repo.repo,
               body: issueBody
             });

      # Step 3: Conditionally add the issue to the project board
      - name: Add 'bug' issue to project board
        if: contains(github.event.issue.labels.*.name, 'bug')
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            github.rest.projects.createCard({
              column_id: 12345678, // Replace with your actual Column ID
              content_id: context.payload.issue.id,
              content_type: "Issue"
            });
```
This final workflow demonstrates the full power of combining these techniques. It creates a comprehensive, templated, and conditional response to new issues, all managed through a clean, multi-step process.