pipeline {
    agent any
    tools {
        jdk 'jdk'
        nodejs 'nodejs22'
    }
    environment {
        SCANNER_HOME = tool 'sonar'
        dockerImageName = 'houyem62/examen'
        dockerImage = ""
    }
    stages {
        stage('Git') {
            steps {
                git branch: 'main', url: 'https://github.com/houyembn/expensse-tracker.git'
            }
        }
         stage("Sonarqube"){
            steps{
                withSonarQubeEnv('sonar') {
                    sh '''  sonar-scanner \ -Dsonar.projectName=examen \
                    -Dsonar.projectKey=examen \
                    '''
                }
            }
        }
        
        
        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build dockerImageName
                }
            }
        }
        stage('Pushing Image') {
            environment {
                registryCredential = credentials('dockerlogin')
            }
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push('latest')
                    }
                }
            }
        }
        stage('Deploy to Kubernetes'){
            steps{
                script{
                    dir('Kubernetes') {
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                                sh 'kubectl apply -f deploymentservice.yml'
                                sh 'kubectl get svc'
                                sh 'kubectl get all'
                        }   
                    }
                }
            }
        }
    }
        
    post {
        always {
            sh 'minikube stop'
        }
    }
    
}
