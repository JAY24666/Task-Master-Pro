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
        stage("Upload Artifacts") {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'nexus:8081',
                    groupId: 'com.master',
                    version: '0.0.1-SNAPSHOT',   // must match POM
                    repository: 'maven-snapshots',  // snapshot repo
                    credentialsId: 'nexus-jenkins-creds',
                    artifacts: [
                     [
                        artifactId: 'todo-app',     // must match POM
                        classifier: '',
                        file: 'target/todo-app-1.0-SNAPSHOT.jar',
                        type: 'jar'
                    ]
                    ]   
                    // artifacts: [
                    //     [artifactId: 'Task-Master-pro',    // must match POM
                    //     classifier: '',
                    //     file: 'target/Task-Master-pro-0.0.1-SNAPSHOT.jar',
                    //     type: 'jar']
                    // ]
                )
            }
        }
        stage('Docker build and Tag') {
            steps {
                script{
                withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh 'docker build -t ash425/taskmaster:latest .'
                    }
                }
            }
        }
        //   stage('Trivy Image scan') {
        //     steps {
        //         sh 'trivy image --format table -o image.html ash425/taskmaster:latest'
        //     }
        // }
        stage('Push Docker Image') {
            steps {
                script{
                withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh 'docker push ash425/taskmaster:latest'
                    }
                }
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name taskmaster -p 8686:80 ash425/taskmaster:latest'
            }
        }
        stage ("Deploy to cluster dev-kt-k8s") {
            steps {
                withKubeConfig(credentialsId: 'kubeconfig-dev-kt-k8s') {
                    sh "kubectl apply -f https://github.com/Ashok220723/Task-Master-Pro/blob/devtask/deployment-service.yml"
                }
            }
        }
    }
}
