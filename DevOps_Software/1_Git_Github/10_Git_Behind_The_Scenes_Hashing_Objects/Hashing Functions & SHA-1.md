#Git #DevOps #Internals #Hashing #SHA1 #Cryptography #GitInternals #CoreConcept

> [!abstract] Brief Description
> This note explains the core concepts of hashing functions and the SHA-1 algorithm. You will learn how cryptographic hash functions guarantee data integrity, why hexadecimal notation is used, and how Git leverages these algorithms to identify files and commits.

---

> [!note] 📖 The Core Analogy: The Digital Fingerprint Scanner
> Imagine a forensic investigator logging evidence into a criminal database:
> - **The Hashing Function (The Fingerprint Scanner):** A machine that takes any piece of evidence—a tiny key, a suitcase, or an entire car—and generates a unique, fixed-size fingerprint card.
> - **Deterministic:** Scanning the exact same finger will *always* produce the exact same fingerprint card.
> - **One-way:** If you steal a copy of the fingerprint card, you cannot reverse-engineer or reconstruct the suspect's actual physical hand from it.
> - **High Avalanche Effect:** Adding a single speck of dust to the finger completely changes the generated fingerprint pattern.
> - **Collision-free:** The odds of two different people generating the exact same fingerprint card are so low they are treated as zero.

---

## 🔢 1. Hashing Functions and Hexadecimal

A **Hashing Function** is a mathematical algorithm that maps input data of arbitrary (variable) size to an output of a fixed size.

### Hexadecimal Notation (Base-16)
The outputs of many hashing functions are represented in **Hexadecimal (Base-16)** notation. Unlike the decimal system (Base-10), which uses digits `0` through `9`, hexadecimal uses sixteen distinct symbols:

$$\text{Symbols: } 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, \text{a}, \text{b}, \text{c}, \text{d}, \text{e}, \text{f}$$

Git uses the **SHA-1** hashing algorithm, which produces a hexadecimal output that is **exactly 40 characters long** (representing a 160-bit number).

---

## 🔒 2. Constraints of Cryptographic Hash Functions

Git relies on a specific subset of hashing algorithms called **Cryptographic Hash Functions**, which must satisfy four strict mathematical constraints:

1.  **Deterministic:** The same input must always produce the exact same output. If you hash the word `"hello"` a million times, you will get the exact same 40-character checksum every time.
2.  **One-Way (Pre-image Resistance):** Given a hash output, it is computationally impossible to reverse-engineer it to determine the original input. 
3.  **High Avalanche Effect (Change Sensitivity):** A minor adjustment to the input (like changing a capital letter to lowercase or adding a single space) must result in a radically different and uncorrelated output.
4.  **Collision-Resistant:** It must be highly improbable to find two different inputs that produce the same output. While mathematical collisions are theoretically possible, the probability is negligible.

---

## 🔑 3. Git's Choice: The SHA-1 Algorithm

Git uses SHA-1 (Secure Hash Algorithm 1) to generate identifiers for commits, trees, blobs, and tags.

```text
# Example of a SHA-1 hash in Git
ce013621a556852fc7e53b53ece5c28b7e5c28b
```

### Hashing as an Integrity Guard
Because Git hashes everything, it does not need to read entire files to check if they have changed. Git simply hashes the file's current state and compares it to the previous hash. If the hashes match, the file is unchanged. If they differ, Git knows the file was modified.

> [!important] The Future of SHA-1 in Git
> In cryptography, SHA-1 is no longer considered secure against modern collision attacks. While Git maintainers plan to transition the database model to **SHA-256** (a stronger 64-character algorithm) in the future, SHA-1 remains the active, default standard for most current Git repositories due to the massive scale of existing repositories.

---

> [!summary] Key Takeaways
> - **Core concept:** Hashing functions map arbitrary inputs to fixed-length hexadecimal outputs, serving as unique identifiers for data blocks.
> - **Key implementation detail:** Git relies on SHA-1 to output 40-character hexadecimal hashes for every object in the database.
> - **Best practice:** Leverage the deterministic and collision-resistant nature of hashes to guarantee that your project history has not been tampered with or corrupted.
