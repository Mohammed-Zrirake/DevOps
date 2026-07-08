#DevOps #IaC #Terraform #CoreConcept #HandsOn #Tutorial #HCL #Security #BestPractice

>  This is a deep dive into mastering [[Terraform Input Variables|Terraform input variables]]. You will learn the best practices for managing variable values with **`.tfvars` files**, how to handle **sensitive data** securely using environment variables, how to define explicit **variable types**, and how to enforce configuration correctness with powerful **custom validation rules**.

---

## 🗂️ Part 1: Managing Variables with `.tfvars` Files

Using `-var` on the command line is cumbersome. The better, more scalable solution is to use **input variable files**.

### Automatically Loaded Variable Files (`terraform.tfvars` & `*.auto.tfvars`)
By default, Terraform automatically loads variable values from files with two specific naming patterns, if they exist:
1.  `terraform.tfvars`
2.  Any file ending in `.auto.tfvars` (e.g., `defaults.auto.tfvars`).

These files are excellent for storing values that are **common across all your environments**.

> [!best-practice] Pick One Pattern
> To keep things simple, it's best to choose one pattern and stick with it. Using a single `terraform.tfvars` file for default values is a common and effective strategy.

**Example:** Instead of a long command line, create a `terraform.tfvars` file:
```hcl
# terraform.tfvars
application_name = "Marks-Blog"
```

### Environment-Specific Variable Files (e.g., `dev.tfvars`)
Some variables, like `environment_name`, are specific to an environment. The best practice is to create a separate `.tfvars` file for each environment.

-   **Important:** These files are **not** automatically loaded. You must explicitly tell Terraform to use them with the `-var-file` command-line option.

**Example Project Structure:**
```
.
├── main.tf
├── variables.tf
├── defaults.auto.tfvars  # Contains common variables like application_name
└── env/
    ├── dev.tfvars        # Contains 'environment_name = "dev"'
    ├── test.tfvars       # Contains 'environment_name = "test"'
    └── prod.tfvars       # Contains 'environment_name = "prod"'
```

**How to run:**
```bash
# To apply the 'dev' environment configuration
terraform apply -var-file="env/dev.tfvars"
```
This provides a powerful and clean way to manage configuration for any number of environments.

### The Order of Operations for Variable Precedence
Terraform loads variable values in a specific order of priority. The last value loaded wins.

1.  **Environment Variables (Highest Priority):** Variables set via `TF_VAR_` environment variables will override all other methods.
2.  **`-var` and `-var-file` Flags:** Values passed in on the command line are next.
3.  **`*.auto.tfvars` Files:** Files ending in `.auto.tfvars` are loaded next, in alphabetical order.
4.  **`terraform.tfvars` File:** This file is loaded after environment variables.
5.  **Variable Defaults:** The `default` value in a `variable` block is used if no other value is provided.
6.  **Interactive Input (Lowest Priority):** If a variable has no value after all the above, Terraform will prompt you to enter one manually.

> [!danger] Avoid Overlapping Definitions
> As a best practice, **only set a given input variable using one technique**. Trying to get fancy by setting the same variable in multiple places (e.g., in a `.tfvars` file and as an environment variable) leads to confusion and makes your configuration hard to debug. It's better for a build to fail due to a missing variable than to succeed with an incorrect, unexpectedly overridden value.

---

## 🛡️ Part 2: Handling Sensitive Data

Sometimes you need to pass sensitive values (secrets, API keys) into your Terraform configuration.

### The Problem: Secret Leakage
-   **Never store secrets in `.tfvars` files.** These files are meant to be committed to source control, and you should never commit secrets.
-   Terraform's default behavior of logging everything to the console can also lead to secrets being exposed in your CI/CD logs.

### The `sensitive` Argument
Mark a variable or output as `sensitive = true` to prevent Terraform from showing its value in console output.
```hcl
# variables.tf
variable "api_key" {
  type      = string
  sensitive = true
}
```
**Effect:**
-   Interactive prompts will hide your typing.
-   Values in `plan` and `apply` logs will be redacted as `(sensitive)`.

> [!warning] **`sensitive` is for Redaction, NOT Encryption!**
> -   `terraform output <sensitive_output>` **will still display the secret** in plain text.
> -   Sensitive values are stored in **plain text** inside the `terraform.tfstate` file.

### The Secure Solution: Environment Variables
The best practice for passing secrets to Terraform in an automated way is through **environment variables**. Terraform has a specific, opinionated naming convention for this:

-   **Format:** `TF_VAR_<variable_name>`
-   The variable name in the environment variable must be an **exact, case-sensitive match** for the name in your `variable` block.

**Example:** To set the `api_key` variable:

-   **PowerShell:** `$env:TF_VAR_api_key = "my-secret-value"`
-   **Bash (Linux/macOS):** `export TF_VAR_api_key="my-secret-value"`

When you run `terraform apply`, Terraform will automatically detect this environment variable and use its value for the `api_key` input, without prompting you or requiring a `-var` flag. This is the most secure way to inject secrets from a CI/CD system's secret management store.

---

## 🏷️ Part 3: Input Variable Types

While Terraform is flexible with types, it's a strong best practice to explicitly define the `type` for each variable. This provides a single source of truth and acts as in-code documentation.

### Primitive Types
-   `string`: A sequence of text characters.
-   `number`: A numeric value.
-   `bool`: A boolean value (`true` or `false`).

### Collection Types
-   `list(<TYPE>)`: An ordered sequence of elements of the same type. `list(string)` is a list of strings. Duplicates are allowed.
-   `map(<TYPE>)` : A collection of key-value pairs, where keys are strings and values are of the same type. `map(number)` is a dictionary mapping strings to numbers.
-   `set(<TYPE>)`: An unordered collection of unique elements of the same type. `set(string)` is a set of unique strings.

### The `object` Type (Complex Objects)
This allows you to define a structured data type with its own set of attributes and their types.
```hcl
variable "sku_settings" {
  type = object({
    kind = string
    tier = string
  })
}
```
**Providing a value in `.tfvars`:**
```hcl
# prod.tfvars
sku_settings = {
  kind = "P"
  tier = "Business"
}
```
**Accessing attributes:** `var.sku_settings.kind`

> [!danger] Be Cautious with Complex Objects
> While powerful, creating deeply nested objects can make your configuration hard to read and manage. Use them judiciously to group tightly coupled, related settings.

---

## ✅ Part 4: Custom Input Variable Validation

To prevent users from providing invalid values, you can add custom `validation` blocks to your variables. This is a powerful way to enforce business logic and constraints.

-   **`condition`:** A required boolean expression. If the expression evaluates to `true`, the validation passes. If it's `false`, Terraform will stop and display the `error_message`.
-   **`error_message`:** A required string that is shown to the user if the condition fails.

**Example: A simple length check.**
```hcl
variable "application_name" {
  type = string
  validation {
    condition     = length(var.application_name) < 12
    error_message = "The application name must be less than 12 characters."
  }
}
```

### Advanced Validation
Validation blocks can contain powerful expressions and, as of Terraform 1.9.0+, can reference other variables, locals, and data sources.

**Example: Enforcing a quorum-based instance count.**
```hcl
# In a central file like validation.tf
locals {
  min_nodes = 5
  max_nodes = 9
}

# In variables.tf
variable "instance_count" {
  type = number
  validation {
    condition = (
      var.instance_count >= local.min_nodes &&
      var.instance_count <= local.max_nodes &&
      var.instance_count % 2 != 0
    )
    error_message = "Instance count must be an odd number between ${local.min_nodes} and ${local.max_nodes}."
  }
}
```
This validation rule ensures the provided instance count is within a defined range *and* is an odd number, preventing a common misconfiguration for quorum-based systems. This is an incredibly powerful feature for creating robust, self-defending modules.