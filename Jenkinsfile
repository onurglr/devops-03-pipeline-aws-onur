pipeline {
    agent {

        label 'My-Jenkins-Agent'

    }

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven 'Maven3'
        jdk 'Java21'
    }

    stages {
        stage('SGM Github') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/onurglr/devops-03-pipeline-aws']])
            }
        }
        stage('Build Maven') {
            steps {
                script{
                if (isUnix()){
                    sh'mvn clean install'
                }else{  bat 'mvn clean install'}
            }
            }
        }
        stage('Test Maven') {
            steps {
                script{
               if (isUnix()){
                    sh'mvn test'
                }else{  bat 'mvn test'}
            }
            }
        }

             stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonar-token') {
                        if (isUnix()) {
                            // Linux or MacOS
                            sh "mvn sonar:sonar"
                        } else {
                            bat 'mvn sonar:sonar'  // Windows
                        }
                    }
                }
            }
        }

        /*
        stage('Docker Image') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'dockerhub_token', variable: 'dockerhub')]) {
                        if (isUnix()) {
                            sh   'docker login -u onurguler18 -p %dockerhub%'
                            sh 'docker push onurguler18/devops-application:latest'
                         }else {
                            bat   'docker login -u onurguler18 -p %dockerhub%'
                            bat 'docker push onurguler18/devops-application:latest'
                        }
                    }
                }
            }
        }

        stage('Deploy Kubernetes') {
            steps {
                kubernetesDeploy(configs: 'deployment-service.yaml', kubeconfigId: 'kubernetes')
            }
        }

        stage('Docker  Image Clean') {
            steps {
            script{
              if (isUnix()){
                    sh 'docker image prune -f'
                }else{   bat 'docker image prune -f'}
               }
            }
        }
*/

    }
}
