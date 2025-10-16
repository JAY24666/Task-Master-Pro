pipeline {
    agent any
    // tools {
    //     maven 'maven3'
    // }
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Kajjayamsriram/Task-Master-Pro.git'
            }
        }
        stage('Bulding App') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Trivy FS scan') {
        steps {
        sh '''
            docker run --rm \
            -v $(pwd):/project \
            aquasec/trivy:latest \
            fs --format table -o /project/fs.html /project
        '''
            }
        }
        // stage('Trivy FS scan') {
        //     steps {
        //         sh 'trivy fs --format table -o fs.html .'
        //     }
        // }     
        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''  $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=blogging -Dsonar.projectKey=blogging \
                    -Dsonar.java.binaries=target '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Publish Artifacts') {
            steps {
                withMaven(globalMavenSettingsConfig: 'settings-maven', jdk: 'jdk17', maven: 'maven3', traceability: true) {
                        sh 'mvn deploy'
                }
            }
        }
    }
}
