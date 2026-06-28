# 🚀 Docker Microservices — Automated CI/CD Pipeline

> A fully automated CI/CD pipeline for a containerized microservices application, covering every stage from code commit to live deployment using industry-standard DevOps tooling.

---

## 🔄 Pipeline Flow

```
Git Push → SonarQube → Nexus → Docker Build → Trivy Scan → Deploy → SMTP Notification
```

---

## 📸 Project Snapshots

| # | Stage | Result |
|:---:|:---|:---|
| 1 | SonarQube Quality Gate | ✅ Passed — All conditions met |
| 2 | Nexus Artifact Storage | `vprofile-v2.war` — 46.2 MB stored |
| 3 | App — Login Page | Running on port `2222` |
| 4 | App — Sign Up Page | Full registration flow working |
| 5 | App — User Profile | Data persisted in DB |
| 6 | App — Social Feed | `devika` profile active |
| 7 | Email Notification | BUILD SUCCESS — prod-deployment `#9` |

### Jenkins Pipeline

![Jenkinspipeline-1](screenshots/Jenkinspipeline-1.png)
![Jenkinspipeline-2](screenshots/Jenkinspipeline-2.png)

### SonarQube Quality Gate

![Sonarqube](screenshots/Sonarqube.png)

### Nexus Artifact Storage

![Nexus](screenshots/Nexus.png)

### Application Pages

![LoginPage](screenshots/LoginPage.png)
![Signuppage](screenshots/Signuppage.png)
![Profilesetup](screenshots/Profilesetup.png)
![Profilefeed](screenshots/Profilefeed.png)

### Post Build Notification

![Postbuildactions](screenshots/Postbuildactions.png)

---

## 🧱 Architecture Overview

This project deploys a **full-stack social/profile web application** as two separate Docker services:

| Service | Image | Role |
|:---|:---|:---|
| App | `mydockerproject:app` | Java/Tomcat web application (WAR) |
| DB | `mydockerproject:db` | MySQL/MariaDB database |

Both images are orchestrated using **Docker Stack**:

```bash
docker stack deploy -c compose.yml mystack
```

---

## ⚙️ Jenkins Pipeline Stages

### Stage 1 — Code Checkout

Clones the application source from the `main` branch of the GitHub repository.

```groovy
git branch: 'main', url: 'https://github.com/JTD-Devops/microservicesproject-docker.git'
```

---

### Stage 2 — Code Quality Analysis (SonarQube)

Static code analysis is performed using SonarQube to detect bugs, vulnerabilities, code smells, and coverage gaps before any build artifact is produced.

```groovy
withSonarQubeEnv("mysonar") {
    sh "mvn clean verify sonar:sonar -Dsonar.projectKey=myproject"
}
```

| Metric | Value |
|:---|:---|
| Quality Gate | ✅ Passed — All conditions met |
| Lines Scanned | 3,300+ |
| Bugs | 34 — Reliability: `E` |
| Vulnerabilities | 2 — Security: `D` |
| Security Hotspots | 13 — `0.0%` reviewed |
| Code Smells | 131 — Technical Debt: `2d 1h` |
| Coverage | 6.7% on 456 lines — 9 unit tests |
| Duplications | 4.8% across 3.3k lines |

> The Quality Gate **passed** — confirming the project meets the minimum thresholds configured in SonarQube before proceeding to build.

---

### Stage 3 — Maven Build

Compiles the Java source code and packages it as a WAR file. The output artifact is copied into the Docker app directory for image assembly.

```groovy
sh "mvn clean package"
sh "cp -r target Docker-app"
```

---

### Stage 4 — Nexus Artifact Storage

The compiled WAR artifact is versioned and uploaded to Nexus Repository Manager for long-term storage, traceability, and potential rollback use.

```groovy
nexusArtifactUploader artifacts: [[
    artifactId: 'vprofile',
    classifier: '',
    file: 'target/vprofile-v2.war',
    type: 'war'
]], credentialsId: '<nexus-credentials>'
```

| Property | Value |
|:---|:---|
| Repository | `myrepo` |
| Format | `maven2` |
| Component Group | `com.visualpathit` |
| Component Name | `vprofile` |
| Version | `v2` |
| File | `vprofile-v2.war` |
| Size | `46.2 MB` |
| Content Type | `application/java-archive` |
| Stored | `Sat May 30 2026 — 07:52:23 IST` |

---

### Stage 5 — Docker Image Build

Two Docker images are built — one for the Java web application and one for the MySQL/MariaDB database — using the Dockerfiles and artifacts prepared in earlier stages.

```groovy
sh "docker build -t devikadockhub/mydockerproject:app Docker-app"
sh "docker build -t devikadockhub/mydockerproject:db Docker-db"
```

---

### Stage 6 — Security Scan (Trivy)

Both images are scanned with Trivy to surface known CVEs at the OS and application layer before they reach the registry or production environment.

```groovy
sh "trivy image devikadockhub/mydockerproject:app"
sh "trivy image devikadockhub/mydockerproject:db"
```

| Finding | Detail |
|:---|:---|
| OS-level CVEs | 69 surfaced on base image — documented |
| Tomcat Core JARs | Majority at 0 CVEs |
| Action Taken | All results reviewed before promoting to deploy |

> Trivy results are reviewed and documented. No critical application-layer CVEs were found in the Tomcat JARs.

---

### Stage 7 — Push to Docker Hub

After scanning, both images are authenticated and pushed to Docker Hub under the `devikadockhub` account for use by the deployment step.

```groovy
withDockerRegistry(credentialsId: '<docker-credentials>') {
    sh "docker push devikadockhub/mydockerproject:app"
    sh "docker push devikadockhub/mydockerproject:db"
}
```

---

### Stage 8 — Deploy via Docker Stack

A **manual approval gate** is inserted before deployment, requiring an explicit human confirmation before the application is promoted to production. Once approved, Docker Stack deploys both services.

```groovy
input message: 'can i deploy the app?'   // Manual approval gate
sh "docker stack deploy -c compose.yml mystack"
```

> 💡 The manual gate ensures no unintended production releases — every deployment is a deliberate, reviewed action.

---

### Stage 9 — SMTP Email Notification

After every pipeline run — whether success or failure — Jenkins automatically sends an email notification with the build result and a direct log URL. Zero manual monitoring is required.

```groovy
post {
    always {
        mail to: 'juttudevika2002@gmail.com',
             subject: "Build Status: ${currentBuild.fullDisplayName}",
             body: "The build ${env.BUILD_NUMBER} finished with status: ${currentBuild.currentResult}. view log at : ${env.BUILD_URL}"
    }
}
```

---

## 🖥️ Application Features (Live)

The deployed application is a full-stack social profile platform with the following features fully operational post-deployment:

| Feature | Status |
|:---|:---|
| User Registration / Sign Up | ✅ Functional |
| Login / Logout | ✅ Functional |
| Update User Profile | ✅ Functional |
| Social Feed / Posts | ✅ Functional |
| Database Persistence | ✅ Verified |
| Password Hashing | ✅ Confirmed (bcrypt) |

> App accessible on port `2222` post-deployment.

---

## 🛠️ Tech Stack

| Tool | Role |
|:---|:---|
| Jenkins | CI/CD orchestration (declarative pipeline) |
| SonarQube | Static code analysis and quality gate |
| Nexus Repository | Artifact versioning and storage |
| Docker | Containerization (App + DB) |
| Docker Hub | Container registry |
| Trivy | Container image security scanning |
| Maven | Build tool |
| Git / GitHub | Source control |
| Amazon Linux | Host OS (EC2) |
| SMTP (Gmail) | Build notification |

---

## 📁 Repository Structure

```
microservicesproject-docker/
├── Docker-app/          # App Dockerfile + WAR
├── Docker-db/           # DB Dockerfile + init scripts
├── compose.yml          # Docker Stack compose file
├── pom.xml              # Maven build config
├── Jenkinsfile          # Declarative pipeline definition
├── screenshots/         # Project snapshot images
└── src/                 # Application source code
```

---

## 🚦 Prerequisites

Before running this pipeline, ensure the following are configured:

| Requirement | Detail |
|:---|:---|
| Jenkins Plugins | SonarQube Scanner, Nexus Artifact Uploader, Docker Pipeline, Email Extension |
| SonarQube | Server running and configured as `mysonar` |
| Nexus | Repository Manager with a `myrepo` maven2 repository |
| Docker | Installed on Jenkins agent (label: `prod`) |
| Docker Hub | Account with push credentials stored in Jenkins |
| Trivy | Installed on Jenkins agent |
| SMTP | Gmail credentials configured in Jenkins |

---

## 📬 Build Notifications

Jenkins sends an automated email after every pipeline run:

| Field | Value |
|:---|:---|
| To | `juttudevika2002@gmail.com` |
| Subject | `Build Status: prod-deployment #9` |
| Body | `The build 9 finished with status: SUCCESS.` |
| Log URL | `http://<jenkins-host>:8080/job/prod-deployment/9/` |

> Triggered automatically on every build — zero manual monitoring required.

---

## 🏆 Key Achievements

| # | Achievement | Metric |
|:---:|:---|:---|
| ✅ | Pipeline stages automated | 8 stages — 0 manual steps |
| ✅ | SonarQube quality gate | PASSED — all conditions met |
| ✅ | Code lines scanned | 3,300+ lines analyzed |
| ✅ | Artifact stored in Nexus | 46.2 MB WAR — SHA1 + MD5 verified |
| ✅ | Docker images built | 2 images (App + DB) — 262.1 MB total |
| ✅ | Images pushed to Docker Hub | Under 1 minute |
| ✅ | CVEs surfaced by Trivy | 69 OS-level CVEs documented |
| ✅ | Tomcat core JARs | Majority at 0 CVEs |
| ✅ | Unit tests executed | 9 tests — 6.7% coverage |
| ✅ | Technical debt tracked | 2 days 1 hour |
| ✅ | Build notification | Auto SMTP email on every build |
| ✅ | Password security | bcrypt hashed — DB verified post-deploy |
| ✅ | Final build status | prod-deployment → SUCCESS |
