# 🔐 Trivy Security Scan – GitHub Actions Walkthrough

## 📌 Overview

This document explains the execution and results of a **Trivy vulnerability scan** performed in a GitHub Actions CI pipeline.

The pipeline:
- Builds a Docker image
- Scans it using Trivy
- Fails if HIGH/CRITICAL vulnerabilities are found

---

## 🚀 Workflow Summary

### Steps executed:

1. **Checkout Code**
2. **Build Docker Image**
3. **Run Trivy Scan**
4. **Analyze Vulnerabilities**
5. **Fail Pipeline (if issues found)**

---

## 🐳 Step 1: Docker Image Build

```bash
docker build -t myapp:<commit-sha> .
````

### ✅ What happened:

* Docker image successfully built
* Base image used:

```dockerfile
FROM python:3.8-slim
```

### ⚠️ Important:

* This base image is **outdated**
* Based on Debian 12 → contains known vulnerabilities

---

## 🔍 Step 2: Trivy Scan Execution

Trivy scanned the built image:

```bash
trivy image myapp:<commit-sha>
```

### During Scan:

* Downloaded vulnerability database (~90MB)
* Scanned:

  * OS packages (Debian)
  * Python dependencies

---

## 📊 Scan Results Summary

```
Total Vulnerabilities: 22
- CRITICAL: 3
- HIGH: 19
```

### ❌ Pipeline Status:

```
FAILED (exit code 1)
```

---

## 🧩 Vulnerability Breakdown

---

### 1️⃣ OS-Level Vulnerabilities (Debian Packages)

These come from the **base image**, not your code.

#### 🔴 Critical Example:

| Package | CVE ID         | Issue                 |
| ------- | -------------- | --------------------- |
| libssl3 | CVE-2025-15467 | Remote Code Execution |

👉 Risk:

* Attackers can execute arbitrary code
* High security impact

---

#### 🔴 Another Critical:

| Package      | CVE ID        | Issue              |
| ------------ | ------------- | ------------------ |
| libsqlite3-0 | CVE-2025-6965 | Integer truncation |

👉 Risk:

* Data corruption
* Exploitable bugs

---

### ⚠️ Key Insight

> Most vulnerabilities are from the **base image (python:3.8-slim)**

---

### 2️⃣ Python Dependency Vulnerabilities

From `requirements.txt`

---

#### Example:

| Package | Version | CVE            | Issue                   |
| ------- | ------- | -------------- | ----------------------- |
| Flask   | 2.0.0   | CVE-2023-30861 | Session cookie exposure |

---

#### More Issues:

| Package    | Issues                                  |
| ---------- | --------------------------------------- |
| setuptools | 3 vulnerabilities (RCE, Path Traversal) |
| wheel      | Privilege escalation                    |

---

## 📌 Why Pipeline Failed?

In workflow:

```yaml
exit-code: '1'
```

### Behavior:

* If HIGH or CRITICAL vulnerabilities found → ❌ FAIL

✔️ This enforces **security gate in CI/CD**

---

## 🔥 Root Cause

### ❌ Main Problem:

```
Outdated Base Image → python:3.8-slim
```

### ❌ Additional Issues:

* Old Python libraries
* Unpatched OS packages

---

## ✅ Fixes & Improvements

---

### 🥇 Fix 1: Update Base Image

```dockerfile
FROM python:3.12-slim
```

✔️ Benefits:

* Latest security patches
* Reduced vulnerabilities

---

### 🥈 Fix 2: Update Dependencies

Update `requirements.txt`:

```txt
flask==2.3.3
setuptools>=70.0.0
wheel>=0.46.2
```

---

### 🥉 Fix 3: Update OS Packages

```dockerfile
RUN apt-get update && apt-get upgrade -y
```

---

### 🧪 Fix 4: Re-run Pipeline

After updates:

* Vulnerabilities should decrease significantly
* Pipeline should pass

---

## 📈 Expected Outcome After Fix

| Before             | After                 |
| ------------------ | --------------------- |
| 22 vulnerabilities | Reduced significantly |
| Pipeline ❌ FAIL    | Pipeline ✅ PASS       |

---

## 🧠 Key Learnings

* Most container vulnerabilities come from **base images**
* Security should be integrated into CI/CD
* Always:

  * Use latest images
  * Keep dependencies updated
  * Scan continuously

---

## ✅ Conclusion

This pipeline demonstrates:

* Automated container security scanning
* Early detection of vulnerabilities
* Enforcement of secure CI/CD practices
