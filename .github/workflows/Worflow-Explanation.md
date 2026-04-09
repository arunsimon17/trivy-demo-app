# 🔐 Trivy Basic Image Scan Workflow — Full Explanation

---

## 🧾 1. Workflow Name

```yaml
name: Basic_Trivy_Image_Scan
```

👉 This is just the **display name** of your workflow in the GitHub Actions UI.

* Helps identify what the pipeline does
* Here: It indicates **security scanning using Trivy**

---

## 🚀 2. Trigger Section

```yaml
on:
  push:
    branches: [ main ]
```

👉 Defines **when the workflow runs**

### ✔️ Meaning:

* Runs **automatically** whenever code is pushed to the `main` branch

### 💡 Why this is important:

* Ensures every new change is **security scanned immediately**
* Prevents vulnerable code from reaching production

---

## 🧱 3. Jobs Section

```yaml
jobs:
  scan:
```

👉 A workflow can have multiple jobs, but here we have one:

### 🔹 Job Name: `scan`

* Logical unit of work
* Runs independently on a runner

---

## 🖥️ 4. Runner Configuration

```yaml
runs-on: ubuntu-latest
```

👉 Specifies the environment where the job runs

### ✔️ Meaning:

* Uses a **GitHub-hosted Ubuntu VM**
* Comes pre-installed with:

  * Docker
  * Git
  * Basic tools

### 💡 Why Ubuntu?

* Lightweight
* Fast startup
* Best support for CI/CD tools

---

## ⚙️ 5. Steps Section

```yaml
steps:
```

👉 Steps are executed **sequentially inside the job**

---

# 🔍 STEP-BY-STEP BREAKDOWN

---

## 📥 Step 1: Checkout Code

```yaml
- name: Checkout code
  uses: actions/checkout@v4
```

👉 This uses the official **GitHub Actions action**.

### ✔️ What it does:

* Pulls your repository code into the runner
* Without this, your workflow has **no access to source code**

### 💡 Think:

➡️ "Clone repo into CI machine"

---

## 🐳 Step 2: Build Docker Image

```yaml
- name: Build Docker Image
  run: |
    docker build -t myapp:${{ github.sha }} .
```

### ✔️ What happens:

* Builds a Docker image from your project
* Uses your `Dockerfile`

### 🔑 Key Concepts:

#### 🏷️ Tagging:

```bash
myapp:${{ github.sha }}
```

* `${{ github.sha }}` = unique commit ID
* Example:

  ```
  myapp:abc1234
  ```

### 💡 Why use SHA?

* Ensures **traceability**
* Each build is uniquely identifiable

### 🧠 Flow:

```
Code → Dockerfile → Docker Image
```

---

## 🔐 Step 3: Run Trivy Scanner

```yaml
- name: Run Trivy vulnerability scanner
  uses: aquasecurity/trivy-action@master
```

👉 Uses **Trivy**, a popular security scanning tool.

---

# 🧩 Trivy Configuration Explained

---

## 🔍 1. What to Scan

```yaml
image-ref: 'myapp:${{ github.sha }}'
```

👉 Specifies the target image

* Scans the image built in Step 2
* Local image inside runner

---

## ⚙️ 2. Scan Type

```yaml
scan-type: 'image'
```

### ✔️ Options:

* `image` → Docker images ✅
* `fs` → file system
* `repo` → Git repo
* `config` → IaC (K8s, Terraform)

👉 Here: scanning container image

---

## 📊 3. Output Format

```yaml
format: 'table'
```

### ✔️ Output styles:

* `table` → human-readable ✅
* `json` → machine-readable
* `sarif` → GitHub security tab

👉 Here: easy to read in logs

---

## 🚨 4. Severity Filter

```yaml
severity: 'CRITICAL,HIGH'
```

👉 Only reports serious vulnerabilities:

* 🔴 CRITICAL → highest risk
* 🟠 HIGH → major risk

### 💡 Why filter?

* Avoid noise from low/medium issues
* Focus on **production blockers**

---

## ❌ 5. Exit Code (Pipeline Control)

```yaml
exit-code: '1'
```

### ✔️ Meaning:

* If vulnerabilities found → **fail pipeline**

### 💡 Behavior:

| Condition   | Result           |
| ----------- | ---------------- |
| No vulns    | ✅ Success        |
| Vulns found | ❌ Pipeline fails |

👉 This enforces **security gate**

---

## 🩹 6. Ignore Unfixed Vulnerabilities

```yaml
ignore-unfixed: true
```

### ✔️ Meaning:

* Skip vulnerabilities with **no available fix**

### 💡 Why?

* Avoid blocking builds unnecessarily
* Focus only on actionable issues

---

## 🔎 7. Vulnerability Types

```yaml
vuln-type: 'os,library'
```

### ✔️ Scans:

#### 🖥️ OS Packages:

* Ubuntu packages
* Alpine packages
* Example:

  * OpenSSL
  * libc

#### 📦 Libraries:

* Node.js packages
* Python packages
* Java dependencies

---

# 🔁 FULL WORKFLOW FLOW

```
Push to main branch
        ↓
GitHub Actions triggered
        ↓
Checkout code
        ↓
Build Docker image
        ↓
Run Trivy scan
        ↓
Check vulnerabilities
        ↓
✔️ Pass → Continue
❌ Fail → Stop pipeline
```

---

# 🎯 REAL-WORLD PURPOSE

This workflow ensures:

### ✅ Security Automation

* Every commit is scanned automatically

### ✅ Shift Left Security

* Catch vulnerabilities **early in development**

### ✅ CI/CD Integration

* Security becomes part of pipeline

