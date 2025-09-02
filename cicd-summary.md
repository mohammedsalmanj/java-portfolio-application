CI_CD_Pipeline.md
# 🚀 CI/CD Pipeline with Jenkins, SonarQube, Nexus, and Slack

This project demonstrates a **complete CI/CD pipeline** using Jenkins.  
It integrates **unit testing, static analysis, code quality gates, artifact publishing, and Slack notifications**.

---

## 🔧 Jenkins Setup

### Tools Configured
- **Maven:** `MAVEN3.9`
- **JDK:** `JDK17`
- **SonarQube Scanner:** `sonar6.2`

### Credentials
- **SonarQube Token:** `sonartoken`
- **Nexus Login:** `nexuslogin`
- **Slack Integration:** channel `#srespace`

---

## 📜 Pipeline Stages

### 1. **Fetch SCM**
- Clones source code from GitHub:
  ```groovy
  git branch: 'main', url: 'https://github.com/mohammedsalmanj/java-portfolio-application.git'

2. Unit Test

Runs project unit tests:

mvn test

3. Checkstyle

Runs static code style checks.

Generates an HTML report at:
target/site/checkstyle.html

Published to Jenkins using HTML Publisher Plugin.

4. SonarQube Analysis

Runs static code + quality analysis.

Project Key: java-portfolio

Uploads:

Java binaries → target/classes

Unit test results → target/surefire-reports

Checkstyle issues → target/checkstyle-result.xml

Uses SonarQube Token (sonartoken) for authentication.

Dashboard URL:
http://<sonar-server>/dashboard?id=java-portfolio
(SonarQube is exposed via port 80 in this setup.)

5. Quality Gate

Waits for SonarQube Quality Gate result:

✅ Pass → pipeline continues.

❌ Fail → pipeline aborts immediately.

6. Build

Builds and packages the WAR file:

mvn clean install


Artifact generated:
target/portfolio-extended-1.0.0.war

7. Upload to Nexus

Publishes the WAR artifact to Nexus Repository Manager:

Nexus URL: http://172.31.17.0:8081

Repository: portfolio-app-repo

Group ID: com.example

Artifact ID: portfolio-extended

Version: 1.0.${BUILD_NUMBER} (dynamic build version)

8. Slack Notifications

Sends real-time notifications to Slack channel #srespace.

Message colors:

🟢 Green → Build Success

🔴 Red → Build Failed

🔵 Blue → Build Unstable (warnings)

This ensures the SRE team is instantly updated on pipeline status.

✅ Benefits

Automated SCM checkout → test → build → deploy.

Checkstyle + SonarQube enforce code quality.

Quality Gate stops bad code early.

WAR artifacts auto-uploaded to Nexus for versioned deployment.

Slack notifications keep the team updated in real-time.

End-to-end traceability with build numbers.

📊 Pipeline Flow (Diagram)
flowchart TD
    A[Fetch SCM] --> B[Unit Test]
    B --> C[Checkstyle]
    C --> D[SonarQube Analysis]
    D --> E[Quality Gate]
    E -->|Pass| F[Build WAR]
    E -->|Fail| X[Pipeline Aborted]
    F --> G[Upload to Nexus]
    G --> H[Slack Notification]

📝 Jenkinsfile (Full Pipeline)
pipeline {
    agent any

    tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }

    environment {
        SONARQUBE_ENV = 'sonarqubeserver'
        SCANNER_HOME  = tool 'sonar6.2'
    }

    stages {
        stage("Fetch SCM") {
            steps {
                git branch: 'main', url: 'https://github.com/mohammedsalmanj/java-portfolio-application.git'
            }
        }

        stage("Unit Test") {
            steps {
                sh "mvn test"
            }
        }

        stage("Checkstyle") {
            steps {
                sh "mvn checkstyle:checkstyle"
                publishHTML(target: [
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'target/site',
                    reportFiles: 'checkstyle.html',
                    reportName: 'Checkstyle Report'
                ])
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    withCredentials([string(credentialsId: 'sonartoken', variable: 'SONAR_TOKEN')]) {
                        sh """
                            ${SCANNER_HOME}/bin/sonar-scanner \
                              -Dsonar.projectKey=java-portfolio \
                              -Dsonar.projectName="Java Portfolio Application" \
                              -Dsonar.projectVersion=1.0 \
                              -Dsonar.sources=src \
                              -Dsonar.java.binaries=target/classes \
                              -Dsonar.junit.reportsPath=target/surefire-reports \
                              -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml \
                              -Dsonar.login=$SONAR_TOKEN
                        """
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Build") {
            steps {
                sh "mvn clean install"
                archiveArtifacts artifacts: 'target/*.war', allowEmptyArchive: true, fingerprint: true
            }
        }

        stage("Upload to Nexus") {
            steps {
                script {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: '172.31.17.0:8081',
                        groupId: 'com.example',
                        version: "1.0.${env.BUILD_NUMBER}",
                        repository: 'portfolio-app-repo',
                        credentialsId: 'nexuslogin',
                        artifacts: [
                            [artifactId: 'portfolio-extended',
                             classifier: '',
                             file: "target/portfolio-extended-1.0.0.war",
                             type: 'war']
                        ]
                    )
                }
            }
        }
    }

    post {
        success {
            slackSend channel: '#srespace',
                      color: 'good',
                      message: "✅ SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' — Artifact uploaded to Nexus."
        }
        failure {
            slackSend channel: '#srespace',
                      color: 'danger',
                      message: "❌ FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' — Check Jenkins for details."
        }
        unstable {
            slackSend channel: '#srespace',
                      color: 'warning',
                      message: "⚠️ UNSTABLE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'."
        }
    }
}