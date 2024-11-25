pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/fasih6/Ekart.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn clean compile"
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                 sh '''
                        ${SCANNER_HOME}/bin/sonar-scanner -Dsonar.host.url=http://52.207.225.101:9000/ \
                        -Dsonar.login=squ_0942c063933dc8a5120efb2c1cc944eb4560dbeb \
                        -Dsonar.projectName=Ekart -Dsonar.java.binaries=. -Dsonar.projectKey=Ekart
                        
                    '''
            }
        }
        stage('OWASP SCAN') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: ' **/dependency-check-report.xml'
            }
        }
        stage('Build Application') {
            steps {
                sh "mvn clean install -DskipTests"
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '9db0f22a-fe75-412e-8287-f1c5d5857ad4', toolName: 'docker') {
                        sh "docker build -t ekart:latest -f docker/Dockerfile ."
                        sh "docker tag ekart:latest fasih6/ekart:latest"
                        sh "docker push fasih6/ekart:latest"
                    }
                }
            }
        }
        stage('Trigger CD Pipeline') {  // this stage isin CD_Pipeline
            steps {
                build job: "CD_Pipeline", wait: true
            }
        }
        stage('Docker Deploy to Container') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '9db0f22a-fe75-412e-8287-f1c5d5857ad4', toolName: 'docker') {
                        sh "docker run -d --name ekart -p 8070:8070 fasih6/ekart:latest"
                    }
                }
            }
        }
    }
}
