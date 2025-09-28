pipeline {
    agent any
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven 'Maven3'
        jdk 'Java21'
    }
    environment {
        APP_NAME = "devops-03-pipeline-aws-onur"
        RELEASE = "1.0"
        GITHUB_USER = "onurglr"
        DOCKER_USER = "onurguler18"
        DOCKER_LOGIN = "dockerhub_token"
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}.${BUILD_NUMBER}"
        DOCKER_HOST = "tcp://localhost:2375"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }
    stages {
        stage('SGM Github') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/onurglr/devops-03-pipeline-aws-onur']])
            }
        }
        stage('Build Maven') {
            steps {
                script {
                    if (isUnix()) {
                        sh'mvn clean install'
                    } else  {
                        bat 'mvn clean install'
                    }
                }
            }
        }
        stage('Test Maven') {
            steps {
                script {
                    if (isUnix()) {
                        sh'mvn test'
                    } else  {
                        bat 'mvn test'
                    }
                }
            }
        }
        
        /*
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
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonar-token'
                }
            }
        }   
         stage('Docker Image') {
             steps {
             //    sh 'docker build  -t  onurguler18/devops-application:latest   .'
                 bat 'docker build  -t  onurguler18/devops-application:latest  .'
             }
         }

        
        stage('Docker Image To DockerHub') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'onur_id_dockerhub_rwd', variable: 'dockerhub')]) {
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
        */
          stage('Build & Push Docker Image to DockerHub') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_LOGIN) {
                        docker_image = docker.build "${IMAGE_NAME}"
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push("latest")
                    }
                }
            }
        }
        /*
        stage("Trivy Scan") {
            steps {
                script {
                    sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image onurguler18/devops-03-pipeline-aws-onur:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }
        }
*/
        
         stage('Cleanup Old Docker Images') {
            steps {
                script {
                    if (isUnix()) {
                        // Bu repo için tüm image’leri al, tarihe göre sırala, son 3 hariç sil
                        sh """
                            docker images "${env.IMAGE_NAME}" --format "{{.Repository}}:{{.Tag}} {{.CreatedAt}}" \\
                            | sort -r -k2 \\
                            | tail -n +4 \\
                            | awk '{print \$1}' \\
                            | xargs -r docker rmi -f
                        """

                    } else {
                        bat """
                             for /f "skip=3 tokens=1" %%i in ('docker images ${env.IMAGE_NAME} --format "{{.Repository}}:{{.Tag}}" ^| sort') do docker rmi -f %%i
                        """
                    }
                }
            }
        }



        stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://localhost:8080/job/onur-devops-03-pipeline-aws-gitops/buildWithParameters?token=GITOPS_TRIGGER_START'"
                }
            }
        }
        
        /*

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
