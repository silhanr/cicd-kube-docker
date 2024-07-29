pipeline {
    agent any
    tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}

  environment {
      registry = "silhanr/vprofileapp"
      registryCredential = "dockerhub"
  }

  stages {
        
        stage('Fetch code') {
          steps{
              git branch: 'main', url:'https://github.com/silhanr/cicd-kube-docker.git'
          }  
        }
        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Test'){
            steps {
                sh 'mvn test'
            }

        }

        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

      stage('Build App Image') {
        steps {
          script {
            dockerImage = docker.build registry + ":V$BUILD_NUMBER" 
          }
        }

      }

      stage('Upload Image') {
        steps {
          script {
            docker.withRegistry('', registryCredential) {
              dockerImage.push("V$BUILD_NUMBER")
              dockerImage.push("latest")
            }
          }
        }

      }

      stage('Remove Unused docker image') {
        steps {
          sh "docker rmi $registry:V$BUILD_NUMBER"
        }
      }

      stage('Kubernetes Deploy') {
        agent {label 'KOPS'}
        steps {
          sh "helm upgrade --install --force vprofile-stack /home/ubuntu/helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER} -n prod"
        }
      }


    }
}