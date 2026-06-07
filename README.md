**🚀 Docker Microservices — Automated CI/CD Pipeline**
> A fully automated CI/CD pipeline for a containerized microservices application, covering every stage from code commit to live deployment using industry-standard DevOps tooling.
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

> 📂 Add screenshots to a `/screenshots` folder and link using `![Stage](screenshots/filename.png)`
---
🔄 Pipeline Flow
```
Git Push → SonarQube → Nexus → Docker Build → Trivy Scan → Deploy → SMTP Notification
```
---
## 🧱 Architecture Overview

This project deploys a **full-stack social/profile web application** as two separate Docker services:

| Service | Image | Role |
|:---|:---|:---|
| App | `mydockerproject:app` | Java/Tomcat web application (WAR) |
| DB  | `mydockerproject:db`  | MySQL/MariaDB database             |

Both images are orchestrated using **Docker Stack**:

```bash
docker stack deploy -c compose.yml mystack
```
---
⚙️ Jenkins Pipeline Stages
### Stage 1 — Code Checkout
```groovy
git branch: 'main', url: 'https://github.com/JTD-Devops/microservicesproject-docker.git'
```

### Stage 2 — Code Quality Analysis (SonarQube)

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

### Stage 3 — Maven Build
```groovy
sh "mvn clean package"
sh "cp -r target Docker-app"
```

### Stage 4 — Nexus Artifact Storage

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
### Stage 5 — Docker Image Build

```groovy
sh "docker build -t devikadockhub/mydockerproject:app Docker-app"
sh "docker build -t devikadockhub/mydockerproject:db Docker-db"
```
### Stage 6 — Security Scan (Trivy)

> `sh "trivy image devikadockhub/mydockerproject:app"`
> `sh "trivy image devikadockhub/mydockerproject:db"`

| Finding | Detail |
|:---|:---|
| OS-level CVEs | 69 surfaced on base image — documented |
| Tomcat Core JARs | Majority at 0 CVEs |
| Action Taken | All results reviewed before promoting to deploy |

### Stage 7 — Push to Docker Hub
```groovy
withDockerRegistry(credentialsId: '<docker-credentials>') {
    sh "docker push devikadockhub/mydockerproject:app"
    sh "docker push devikadockhub/mydockerproject:db"
}
```
### Stage 8 — Deploy via Docker Stack
```groovy
input message: 'can i deploy the app?'   // Manual approval gate
sh "docker stack deploy -c compose.yml mystack"
```
> 💡 A manual approval gate pauses the pipeline before production deployment — ensuring intentional release control.
Stage 9 — SMTP Email Notification
```groovy
post {
    always {
        mail to: 'juttudevika2002@gmail.com',
             subject: "Build Status: ${currentBuild.fullDisplayName}",
             body: "The build ${env.BUILD_NUMBER} finished with status: ${currentBuild.currentResult}. view log at : ${env.BUILD_URL}"
    }
}
```
Automated email on every build completion (success or failure)
Zero manual monitoring required
---
🖥️ Application Features (Live)
The deployed application is a full-stack social profile platform with:
Feature	Status
User Registration / Sign Up	✅ Functional
Login / Logout	✅ Functional
Update User Profile	✅ Functional
Social Feed / Posts	✅ Functional
Database Persistence	✅ Verified
Password Hashing	✅ Confirmed
> App accessible on port `2222` post-deployment.
---
## 🛠️ Tech Stack

| Tool | Role |
|:---|:---|
| Jenkins | CI/CD Orchestration (Declarative Pipeline) |
| SonarQube | Static Code Analysis & Quality Gate |
| Nexus Repository | Artifact Versioning & Storage |
| Docker | Containerization (App + DB) |
| Docker Hub | Container Registry |
| Trivy | Container Image Security Scanning |
| Maven | Build Tool |
| Git / GitHub | Source Control |
| Amazon Linux | Host OS (EC2) |
| SMTP (Gmail) | Build Notification |
---
📁 Repository Structure
```
microservicesproject-docker/
├── Docker-app/          # App Dockerfile + WAR
├── Docker-db/           # DB Dockerfile + init scripts
├── compose.yml          # Docker Stack compose file
├── pom.xml              # Maven build config
├── Jenkinsfile          # Declarative pipeline definition
└── src/                 # Application source code
```
---
## 🚦 Prerequisites

| Requirement | Detail |
|:---|:---|
| Jenkins Plugins | SonarQube Scanner, Nexus Artifact Uploader, Docker Pipeline, Email Extension |
| SonarQube | Server running and configured as `mysonar` |
| Nexus | Repository Manager with a `myrepo` maven2 repository |
| Docker | Installed on Jenkins agent (label: `prod`) |
| Docker Hub | Account with push credentials stored in Jenkins |
| Trivy | Installed on Jenkins agent |
| SMTP | Credentials configured in Jenkins |
---
## 📬 Build Notifications

Jenkins sends an email after every pipeline run:

| Field | Value |
|:---|:---|
| To | `juttudevika2002@gmail.com` |
| Subject | `Build Status: prod-deployment #9` |
| Body | `The build 9 finished with status: SUCCESS.` |
| Log URL | `http://<jenkins-host>:8080/job/prod-deployment/9/` |

> Triggered automatically on every build — zero manual monitoring required.
```
## 🏆 Key Achievements

| # | Achievement | Metric |
|:---:|:---|:---|
| ✅ | Pipeline Stages Automated | 8 Stages — 0 Manual Steps |
| ✅ | SonarQube Quality Gate | PASSED — All Conditions Met |
| ✅ | Code Lines Scanned | 3,300+ Lines Analyzed |
| ✅ | Artifact Stored in Nexus | 46.2 MB WAR — SHA1 + MD5 Verified |
| ✅ | Docker Images Built | 2 Images (App + DB) — 262.1 MB Total |
| ✅ | Images Pushed to Docker Hub | Under 1 Minute |
| ✅ | CVEs Surfaced by Trivy | 69 OS-level CVEs Documented |
| ✅ | Tomcat Core JARs | Majority at 0 CVEs |
| ✅ | Unit Tests Executed | 9 Tests — 6.7% Coverage |
| ✅ | Technical Debt Tracked | 2 Days 1 Hour |
| ✅ | Build Notification | Auto SMTP Email on Every Build |
| ✅ | Password Security | bcrypt Hashed — DB Verified Post-Deploy |
| ✅ | Final Build Status | prod-deployment → SUCCESS |
