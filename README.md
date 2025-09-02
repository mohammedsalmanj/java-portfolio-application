# java-portfolio-application



Focusing on CICD and making much more crazy analogy to keep in sync

Think of this like building a car in a factory:

Assemble chassis (build/test)

Paint & polish (Checkstyle)

Inspection (Sonar)

Final QC approval (Quality Gate)

Send to warehouse (Nexus)

Boss informed (Slack)



# Java Portfolio CI/CD Pipeline

This repository contains a **Jenkins Declarative Pipeline** for building, testing, analyzing, and deploying a Java web application.  
It integrates with **SonarQube**, **Nexus**, and **Slack** for a complete CI/CD workflow.

---

## 🚀 Pipeline Stages

1. **Fetch SCM**
   - Pulls source code from GitHub repository.

2. **Unit Test**
   - Runs `mvn test` using Maven Surefire plugin.
   - Ensures existing unit tests are executed.

3. **Checkstyle**
   - Runs static code style checks with Maven Checkstyle plugin.
   - Publishes `checkstyle.html` report in Jenkins.

4. **SonarQube Analysis**
   - Runs `sonar-scanner` to analyze source code.
   - Reports code quality, bugs, vulnerabilities, and code smells.
   - Requires `sonartoken` credential in Jenkins.

5. **Quality Gate**
   - Jenkins waits for SonarQube Quality Gate status.
   - Pipeline fails if quality gate is **FAILED**.

6. **Build**
   - Runs `mvn clean install`.
   - Produces a `.war` file under `target/`.
   - Archives artifact in Jenkins.

7. **Upload to Nexus**
   - Publishes the `.war` artifact to **Nexus Repository Manager** (`portfolio-app-repo`).
   - Example artifact coordinates:
     ```
     groupId: com.example
     artifactId: portfolio-extended
     version: 1.0.<BUILD_NUMBER>
     ```

8. **Slack Notification**
   - Sends build status notifications to Slack channel **#srespace**.
   - ✅ Success → Green notification with build & artifact details.  
   - ❌ Failure → Red notification with job & build number.

---

## 🔧 Prerequisites

- **Jenkins Plugins:**
  - Pipeline
  - Maven Integration
  - SonarQube Scanner
  - Nexus Artifact Uploader
  - Slack Notification

- **Configured Tools in Jenkins:**
  - Maven → `MAVEN3.9`
  - JDK → `JDK17`
  - SonarQube Scanner → `sonar6.2`
  - SonarQube Server → `sonarqubeserver`

- **Jenkins Credentials:**
  - `sonartoken` → SonarQube user token
  - `nexuslogin` → Nexus username/password

- **Slack Setup:**
  - Install Slack Notification Plugin.
  - Configure Jenkins with Slack Workspace + Token.
  - Set default channel → `#srespace`.

---

## 📊 Reports & Dashboards

- **SonarQube Dashboard:**  
  Shows code quality, bugs, vulnerabilities, and coverage.  
  Example: `http://<sonar-server>:/dashboard?id=java-portfolio`

- **Checkstyle Report in Jenkins:**  
  `target/site/checkstyle.html`

- **Nexus Repository:**  
  Published artifacts available in:  
  `http://<nexus-server>:8081/repository/portfolio-app-repo/`

- **Slack Notifications:**  
  Build updates in channel **#srespace**

---

## ✅ Example Success Flow

1. Jenkins pulls code from GitHub.  
2. Runs unit tests and static analysis.  
3. Uploads results to SonarQube.  
4. Waits for Quality Gate ✅.  
5. Builds `.war` file and uploads to Nexus.  
6. Sends Slack notification (green ✅) to `#srespace`.

---

## ❌ Example Failure Flow

- If **tests fail**, **Checkstyle fails**, or **SonarQube Quality Gate** fails → pipeline stops.  
- Jenkins sends Slack notification (red ❌) to `#srespace`.

---
