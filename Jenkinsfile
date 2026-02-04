pipeline {
    agent any
    
    tools {
        maven 'Maven'
    }
    
    options {
        skipDefaultCheckout()
    }
    
    stages {
        stage('Checkout Source') {
            steps {
                git url: 'git@github.com:VAISAALI18/devops-app.git',
                    credentialsId: 'github-ssh-key',
                    branch: 'main'
            }
        }
        
        stage('Build') {
            steps {
                echo 'Starting Maven Build'
                sh 'mvn clean compile'
            }
        }
        
        stage('Build and Unit Tests') {
            steps {
                echo 'Running Unit Tests with Coverage'
                sh 'mvn test'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube Analysis'
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('My Sonar Server') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=devops-app \
                            -Dsonar.projectName=devops-app \
                            -Dsonar.sources=src/main/java \
                            -Dsonar.tests=src/test/java \
                            -Dsonar.java.binaries=target/classes
                        """
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                echo 'Waiting for SonarQube Quality Gate result...'
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Package') {
            steps {
                echo 'Packaging Application'
                sh 'mvn package -DskipTests'
            }
        }
        
        stage('Archive Artifacts') {
            steps {
                echo 'Archiving Build Artifacts and Test Results'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                junit 'target/surefire-reports/*.xml'
                echo 'Coverage report available in SonarQube dashboard'
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Pipeline succeeded! Code passed all quality gates and built successfully.'
        }
        failure {
            echo 'Pipeline Failed - Either build failed or Quality Gate was not met.'
        }
    }
}
