# Lecture Plan: Introduction to YAML

**Objective**: By the end of the session, students will understand what YAML is, its syntax (with a focus on indentation), and how to incorporate arrays in YAML files. They will also gain hands-on experience creating and validating YAML files.

---

## 1. Introduction to YAML

**Objective**: Explain what YAML is and its role in DevOps.

**Key Points**:
- **What is YAML?**
  - YAML stands for "YAML Ain’t Markup Language."
  - A human-readable data serialization format used for configuration files in DevOps tools (e.g., Docker, Kubernetes, Ansible, CI/CD pipelines like GitHub Actions).
  - Designed to be simple, concise, and easy to read/write compared to XML or JSON.
  - Commonly used for defining infrastructure as code, CI/CD pipelines, and application configurations.

- **Why YAML in DevOps?**
  - Simplifies configuration management.
  - Widely supported by DevOps tools (e.g., Kubernetes manifests, Ansible playbooks).
  - Human-friendly syntax reduces errors and improves collaboration.

- **Basic Structure**:
  - YAML files use `.yml` or `.yaml` extensions.
  - Data is structured using key-value pairs, arrays (lists/sequences), and nested objects (mappings).
  - Relies heavily on indentation for hierarchy (unlike JSON, which uses braces).

**Example**:
- yaml Example:
  ```yaml
  name: John Doe
  age: 30
  city: New York
  ```
- Compare it briefly with JSON to highlight readability:
  ```json
  {
    "name": "John Doe",
    "age": 30,
    "city": "New York"
  }
  ```
- YAML’s use in DevOps (e.g., a Kubernetes pod definition or GitHub Actions workflow).
---

## 2. Understanding YAML Indentation

**Objective**: Teach the rules of YAML indentation and common pitfalls.

**Key Points**:
- **Indentation Rules**:
  - YAML uses spaces (not tabs) for indentation.
  - Standard indentation is **2 spaces** per level (though any consistent number of spaces works).
  - Indentation defines the hierarchy of data (parent-child relationships).
  - Inconsistent indentation causes parsing errors.

- **Example**:
  ```yaml
  person:
    name: Alice
    age: 25
    address:
      street: 123 Main St
      city: Boston
  ```
  - `person` is the parent.
  - `name`, `age`, and `address` are children (indented 2 spaces).
  - `street` and `city` are children of `address` (indented 4 spaces).

- **Common Pitfalls**:
  - Mixing tabs and spaces (YAML parsers reject tabs).
  - Inconsistent indentation (e.g., 2 spaces in one section, 4 in another).
  - Misaligned keys/values causing incorrect hierarchy.

- **Validation Tools**:
  - Use online YAML validators (e.g., yamllint.com) or CLI tools like `yamllint` to check syntax.
  - Many DevOps tools (e.g., Kubernetes) fail if YAML is malformed, so validation is critical.

**Example**:
- i.e.:
  ```yaml
  # Correct
  server:
    host: localhost
    port: 8080

  # Incorrect (inconsistent indentation)
  server:
      host: localhost
    port: 8080
  ```

---

## 3. Arrays (Lists/Sequences) in YAML (20 minutes)

**Objective**: Demonstrate how arrays are represented in YAML and their practical use cases.

**Key Points**:
- **What are Arrays in YAML?**
  - Called "sequences" in YAML terminology.
  - Represent ordered lists of items (strings, numbers, objects, or other sequences).
  - Denoted by a hyphen (`-`) followed by a space.

- **Types of Arrays**:
  1. **Simple Sequences (List of Scalars)**:
     ```yaml
     fruits:
       - Apple
       - Banana
       - Orange
     ```
     - Each item is a string, prefixed with `-`.

  2. **Sequence of Mappings (List of Objects)**:
     ```yaml
     employees:
       - name: Alice
         role: Developer
       - name: Bob
         role: Tester
     ```
     - Each item is a mapping (key-value pairs) under a sequence.

  3. **Nested Sequences**:
     ```yaml
     teams:
       - name: Team A
         members:
           - Alice
           - Bob
       - name: Team B
         members:
           - Charlie
           - Dave
     ```
     - Arrays can contain other arrays or mappings.

  4. **Inline Sequences**:
     ```yaml
     colors: [Red, Green, Blue]
     ```
     - Compact syntax for simple lists, equivalent to:
       ```yaml
       colors:
         - Red
         - Green
         - Blue
       ```

- **Use Cases in DevOps**:
  - Defining multiple containers in a Docker Compose file:
    ```yaml
    services:
      - name: web
        image: nginx
      - name: db
        image: mysql
    ```
  - Specifying multiple steps in a CI/CD pipeline (e.g., GitHub Actions):
    ```yaml
    jobs:
      build:
        steps:
          - run: npm install
          - run: npm test
    ```

- **Best Practices**:
  - Use consistent indentation (2 spaces) for sequence items.
  - Align hyphens (`-`) vertically for readability.
  - Avoid overly complex nesting to keep YAML files manageable.

---