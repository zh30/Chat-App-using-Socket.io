pipeline {

     environment {
          registry = "diegotc/chat-channel"
          registryCredential = '826db27a-8ac9-41a9-83e8-0a920204a2f0'
     }

     agent any
     stages {
         stage('Build') {
             steps {
                 sh 'echo "Hello World"'
                 sh '''
                     echo "Multiline shell steps works too"
                     ls -lah
                 '''
             }
         }
         stage ("lint dockerfile") {
            agent {
                docker {
                    image 'hadolint/hadolint:latest-debian'
                }
            }

            steps {
                sh 'hadolint dockerfiles/* | tee -a hadolint_lint.txt'
            }

            post {
                always {
                    archiveArtifacts 'hadolint_lint.txt'
                }
            }
         }

         stage('Building image') {
            steps{
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
         }


         stage('Deploy Image') {
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                    dockerImage.push()
                }
            }
         }

    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }

         stage('Security Scan') {
              steps {
                 aquaMicroscanner imageName: 'alpine:latest', notCompliesCmd: 'exit 1', onDisallowed: 'fail', outputFormat: 'html'
              }
         }
     }
}