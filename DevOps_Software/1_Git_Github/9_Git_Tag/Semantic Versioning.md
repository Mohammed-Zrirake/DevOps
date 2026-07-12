#Git #DevOps #Versioning #SemVer #BestPractice #CoreConcept

> [!abstract] Brief Description
> This note explains the specification of **Semantic Versioning (SemVer)**. It defines the meaning behind the `MAJOR.MINOR.PATCH` versioning format, explains the rules for incrementing version numbers, and demonstrates how it works in real-world libraries like React and Bootstrap.

---

> [!note] 📖 The Core Analogy: Street Address Renovations
> Imagine telling your neighbors about changes you are making to your home:
> - **PATCH (Painting the Front Door):** A minor change. Neighbors do not need to change how they navigate your house. (Increment the third digit).
> - **MINOR (Building a Back Deck):** A new feature. Neighbors can still visit you the exact same way, but there is a new optional area to enjoy. (Increment the middle digit, reset patch to 0).
> - **MAJOR (Demolishing and Rebuilding):** A breaking change. The front door is now on the opposite side of the block. Anyone visiting must completely change how they access your home. (Increment the first digit, reset minor and patch to 0).

---

## 📋 1. What is Semantic Versioning?

**Semantic Versioning (SemVer)** is a formal specification that dictates how version numbers are assigned and incremented. Its goal is to make software updates transparent, consistent, and predictable for developers who consume your code as a dependency.

The standard version number format consists of three digits separated by periods:

$$\text{MAJOR}.\text{MINOR}.\text{PATCH}$$

*   **Initial Release:** Under the SemVer specification, the first stable public release of a software API starts at `1.0.0`.

---

## ⚙️ 2. The SemVer Increment Rules

Once `1.0.0` is released, version numbers are incremented based on the nature of the changes made to the codebase:

### 1. PATCH (Bug Fixes)
Increment the third digit (e.g., `1.0.0` $\rightarrow$ `1.0.1`) when you make backwards-compatible bug fixes or minor adjustments. No new features are introduced, and nothing should break for the end-user.

### 2. MINOR (New Features)
Increment the middle digit and reset the patch to zero (e.g., `1.0.1` $\rightarrow$ `1.1.0`) when you add new functionality in a backwards-compatible manner. Users can upgrade safely without rewriting their existing code, but they get access to new optional features.

### 3. MAJOR (Breaking Changes)
Increment the first digit and reset the minor and patch to zero (e.g., `1.1.3` $\rightarrow$ `2.0.0`) when you introduce breaking changes that are not backwards-compatible. Existing integrations may fail, and users must consult release notes or upgrade guides to migrate their code.

---

## 🌍 3. Real-World Applications: React & Bootstrap

Most major frameworks and libraries adhere closely to SemVer to manage their public distributions:

### Bootstrap Example (Patches)
In Bootstrap's release log, upgrading from `4.5.1` to `4.5.2` is a patch release. It contains very minor tweaks (e.g., removing a single line of CSS styling or restoring a Sass mixin) with absolutely no breaking adjustments.

### React Example (Minors and Majors)
*   **Minor Release:** In React, going from `16.8.6` to `16.9.0` represents a minor release. It introduced new features (like deprecation warnings for older lifecycles) but kept everything backwards-compatible, resetting the patch number to `0`.
*   **Major Release:** Going to `17.0.0` represents a major release. It contained breaking architectural updates, requiring the React team to publish comprehensive migration blogs and upgrade guides for developers.

> [!tip] Prefixes and Pre-Releases
> *   **The `v` Prefix:** While not strictly part of the SemVer specification, it is standard practice in Git to prefix version numbers with a `"v"` when creating tags (e.g., `v1.0.0`).
> *   **Pre-Releases:** Pre-releases are marked by appending a hyphen and identifier (e.g., `v1.0.0-alpha`, `v1.0.0-beta.1`).

---

> [!summary] Key Takeaways
> - **Core concept:** Semantic Versioning communicates the scope and compatibility risk of a software update through a three-digit `MAJOR.MINOR.PATCH` sequence.
> - **Key implementation detail:** When incrementing a higher-level digit (e.g., MINOR), all lower-level digits (e.g., PATCH) must be reset back to zero.
> - **Best practice:** Align your project's release versions with SemVer rules to prevent breaking downstream applications that depend on your code.
