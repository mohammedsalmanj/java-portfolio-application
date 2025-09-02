pipeline {
    agent any   // Run on any available Jenkins agent

    tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }

    environment {
        SONARQUBE_ENV = 'sonarqubeserver'   // SonarQube server name from Jenkins config
        SCANNER_HOME  = tool 'sonar6.2'     // SonarQube Scanner tool from Jenkins tool config
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
            post {
                success {
                    echo "✅ Build Success — WAR file created!"
                }
            }
        }

        stage("Upload to Nexus") {
            steps {
                script {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: '172.31.17.0:8081',   // 🔹 Your Nexus IP + port
                        groupId: 'com.example',         // 🔹 Adjust if needed
                        version: "1.0.${env.BUILD_NUMBER}", // dynamic version
                        repository: 'portfolio-app-repo',   // 🔹 Nexus repo name
                        credentialsId: 'nexuslogin',        // 🔹 Jenkins credentials ID
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
}
