pipeline{
   agent {
        node {
            label "java-node"
        }
    }
   environment {
      registryCredential='docker-hub-credentials'
      registryBackend = 'cramosdev/backend-demo'
    }
    
    stages {
        stage('Build Project') {
          steps {
               sh "mvn clean install -DskipTests"
          }
        }

//         stage('SonarQube analysis') {
//           steps {
//             withSonarQubeEnv(credentialsId: "sonarqube-credentials", installationName: "sonarqube-server"){
//                 sh "mvn clean verify sonar:sonar -DskipTests"
//             }
//           }
//         }

//         stage('Quality Gate') {
//             steps {
//                 timeout(time: 10, unit: "MINUTES") {
//                     script {
//                         def qg = waitForQualityGate(webhookSecretId: 'sonarqube-credentials')
//                         if (qg.status != 'OK') {
//                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
//                         }
//                     }
//                 }
//             }
//         }
      
       stage('Push Image to Docker Hub') {
            steps {
                script {
                    dockerImage = docker.build registryBackend + ":latest"
                    docker.withRegistry( '', registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }
       stage("Deploy to K8s"){
           steps{
               script {
                 if(fileExists("configuracion")){
                   sh 'rm -r configuracion'
                 }
               }

               sh 'git clone https://github.com/cramosdev/minikube-training.git configuracion --branch main'
               sh 'kubectl apply -f configuracion/kubernetes-deployment/spring-boot-app/manifest.yml -n default --kubeconfig=configuracion/kubernetes-config/config'
           }
       }

       stage ("Run API Test") {
           steps{
               node("jenkins-nodejs"){
                   script {
                       if(fileExists("spring-qa")){
                           sh 'rm -r spring-qa'
                       }
                       sleep 15 // seconds
                       sh 'git clone https://github.com/cramosdev/spring-qa.git --branch master'
                       sh 'newman run spring-qa/src/test/resources/postman_api_test.json --reporters cli,junit --reporter-junit-export "newman/report.xml"'
                       junit "newman/report.xml"
                   }
               }
           }
       }

    }
    
    post {
        always {
            sh "docker logout"
            sh "docker rmi -f " + registryBackend + ":latest"
        }
    }  
}
