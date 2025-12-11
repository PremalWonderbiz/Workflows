# 📘 **Generic CI Pipeline — Documentation**

This repository contains a reusable GitHub Actions workflow named **`generic-ci.yml`**.
It provides a **complete CI pipeline** for multi-project repositories consisting of:

* **Backend (.NET)** services
* **Frontend (Node/React/Next.js)** projects

The workflow performs:

✔ Change detection
✔ Configuration validation
✔ SonarQube analysis
✔ Coverity static analysis
✔ SBOM generation
✔ CI summary report

It is triggered using `workflow_call`, allowing other workflows to invoke it.

---

## 🚀 **1. How to Use This Workflow**

Add the following inside any workflow to call **generic-ci.yml**:

```yml
jobs:
  run-ci:
    uses: ./.github/workflows/generic-ci.yml
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      COVERITY_BACKEND_PROJECT_TOKEN: ${{ secrets.COVERITY_BACKEND_PROJECT_TOKEN }}
      COVERITY_FRONTEND_PROJECT_TOKEN: ${{ secrets.COVERITY_FRONTEND_PROJECT_TOKEN }}
    with:
      backend_sonar_key: "<your-backend-sonar-key>"
      frontend_sonar_key: "<your-frontend-sonar-key>"
      SONAR_HOST_URL: "https://sonarcloud.io"
      SONAR_ORG: "<your-organization>"
      COVERITY_EMAIL: "<notification-email>"
```

---

## 🔐 **2. Required Inputs & Secrets**

### **Inputs**

| Name                 | Required | Description                                |
| -------------------- | -------- | ------------------------------------------ |
| `backend_sonar_key`  | ✔        | SonarQube key for backend                  |
| `frontend_sonar_key` | ✔        | SonarQube key for frontend                 |
| `SONAR_HOST_URL`     | ✔        | SonarQube host URL                         |
| `SONAR_ORG`          | ✔        | Sonar organization                         |
| `COVERITY_EMAIL`     | ✔        | Email used when uploading Coverity results |

### **Secrets**

| Secret                            | Required | Purpose                        |
| --------------------------------- | -------- | ------------------------------ |
| `SONAR_TOKEN`                     | ✔        | SonarCloud authentication      |
| `COVERITY_BACKEND_PROJECT_TOKEN`  | ✔        | Backend Coverity upload token  |
| `COVERITY_FRONTEND_PROJECT_TOKEN` | ✔        | Frontend Coverity upload token |

---

## 🧩 **3. Pipeline Overview**

The workflow contains **6 major jobs**:

1. **detect-changes**
2. **load-validate-configuration**
3. **sonar-analysis**
4. **coverity-analysis**
5. **sbom-analysis**
6. **github-summary**

Each step is explained below.

---

# 📝 **4. Job Details**

---

## **4.1 🔍 detect-changes**

Purpose:
Detect which projects changed in the last commit.

Key features:

* Uses `git diff` to list changed files
* Reads `Configurations/project-config.yml`
* Determines:

  * `backend_changed = true/false`
  * `frontend_changed = true/false`

Outputs used by next job to decide whether to skip analysis.

---

## **4.2 📦 load-validate-configuration**

Purpose:
Load configuration and validate required settings.

### Responsibilities:

* Loads project lists from:
  `Configurations/project-config.yml`
* Loads backend/frontend settings:

  * sonar
  * sbom
  * coverity
* Validates presence of required config files
* Applies skip logic:

  * Skip sonar if disabled or unchanged
  * Skip coverity if project name missing or unchanged
  * Skip SBOM if disabled or unchanged

### Outputs:

| Output                        | Description                    |
| ----------------------------- | ------------------------------ |
| `skipBackendSonar`            | skip backend sonar?            |
| `skipFrontendSonar`           | skip frontend sonar?           |
| `skipBackendCoverity`         | skip backend coverity?         |
| `skipFrontendCoverity`        | skip frontend coverity?        |
| `skipBackendSbom`             | skip backend SBOM?             |
| `skipFrontendSbom`            | skip frontend SBOM?            |
| `backendServices`             | list of backend service paths  |
| `frontendProjects`            | list of frontend project paths |
| `backendCoverityProjectName`  | Coverity backend project       |
| `frontendCoverityProjectName` | Coverity frontend project      |

---

# **4.3 🔎 sonar-analysis**

Runs **SonarQube** analysis for backend & frontend.

### Backend (C#, .NET)

Steps:

* Install .NET, Node, Java
* Install `dotnet-sonarscanner`
* Auto-build inclusion paths from config
* Begin SonarQube analysis
* Build & test each backend service
* End Sonar analysis

### Frontend (Node)

Steps:

* Install dependencies
* Build MFEs
* Run tests (if available)
* Dynamically update `sonar-project.properties`
* Run `sonar-scanner`

---

# **4.4 🛡️ coverity-analysis**

Runs **Coverity Static Analysis** for backend & frontend.

### Backend

* Download Coverity tools (token + project name)
* Build all backend services using Coverity
* Package reports into `cov-int.tgz`
* Upload results to Coverity Scan

### Frontend

* Uses **Coverity Buildless Capture**
* Captures JS/TS code
* Excludes:

  * node_modules
  * .next, dist, coverage
  * test files

---

# **4.5 📦 sbom-analysis**

Generates **Software Bill of Materials (SBOM)** for backend & frontend.

Uses `sbom-tool`:

### Backend

* Build output for each `.sln`
* Generate SBOM under `sbom/<service-name>/`

### Frontend

* Install dependencies
* Build project
* Copy built output (.next / dist)
* Generate SBOM

Uploads SBOM artifacts at the end.

---

# **4.6 🧾 github-summary**

Generates a **beautiful CI summary** in GitHub Actions:

Includes:

* SonarQube backend & frontend links
* Coverity backend & frontend links
* SBOM generation note

Displayed under the **Summary** tab.

---

# 🗂️ **5. Folder Structure Requirements**

```
Configurations/
│── project-config.yml
│── sonar/
│    ├── SonarQube.Analysis.xml
│    └── FrontendSonarAnalysis/sonar-project.properties
```

---

# 📄 **6. Configuration File Format**

Your `Configurations/project-config.yml` should define:

### Example:

```yml
backendServices:
  - name: ServiceA
    path: Backend/ServiceA

frontendProjects:
  - name: WebApp
    path: Frontend/WebApp

settings:
  backend:
    sonar: true
    coverity: true
    sbom: true
    coverity_project_name: "backend-project"
  frontend:
    sonar: true
    coverity: false
    sbom: true
```

---

# 🧠 **7. Key Features Summary**

| Feature                          | Supported             |
| -------------------------------- | --------------------- |
| Auto-detect project changes      | ✅                     |
| Auto-skip jobs when not required | ✅                     |
| Backend Sonar (C#)               | ✅                     |
| Frontend Sonar (Node)            | ✅                     |
| Coverity Analysis                | 🔹 Backend & Frontend |
| SBOM generation                  | Backend + Frontend    |
| Central configuration            | Yes                   |

---

# 🎯 **8. Advantages of This CI**

* Reusable across multiple repos
* Fully configurable via YAML
* Efficient execution due to skip logic
* Supports multi-backend & multi-frontend architectures
* Uplifts security compliance (Coverity + SBOM)
* Ensures code quality via SonarQube
