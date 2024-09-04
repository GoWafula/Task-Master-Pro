pipeline {
    agent any

    tools {
        maven 'maven3'  // Ensure 'maven3' is defined in Global Tool Configuration
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'  // Ensure 'sonar-scanner' is defined in Global Tool Configuration
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/GoWafula/Task-Master-Pro.git'
            }
        }

        stage('Compilation') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Test') {
            steps {
                sh "mvn test"
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html ."
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {  // Ensure 'sonar' is configured in Jenkins' SonarQube settings
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=Taskmaster \
                        -Dsonar.projectName=Taskmaster \
                        -Dsonar.java.binaries=target
                    '''
                }
            }
        }

        stage('Build Application') {
            steps {
                sh "mvn package"
            }
        }

        stage('Publish on Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'settings', maven: 'maven3') {
                    sh "mvn deploy"
                }
            }
        }

        stage('Build and Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker build -t wafula101/taskmaster:latest ."
                    }
                }
            }
        }

        stage('Scan Docker Image with Trivy') {
            steps {
                sh "trivy image --format table -o image-report.html wafula101/taskmaster:latest"
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push wafula101/taskmaster:latest"
            }
        }
    }
}
