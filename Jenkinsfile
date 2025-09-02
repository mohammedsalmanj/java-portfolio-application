q

pipeline {
    agent any   // simpler than agent { label "any" }
    tools {
        maven "MAVEN3.9"
        jdk "JDK17"
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

        stage("Build") {
            steps {
                sh "mvn clean install -DskipTests"
            }
            post {
                success {
                    archiveArtifacts artifacts: '**/target/*.war', fingerprint: true
                    echo "✅ Build Success"
                }
                failure {
                    echo "❌ Build failed"
                }
            }
        }
    }
}
