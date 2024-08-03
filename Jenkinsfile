pipeline {

    agent any

        tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }

    environment {
                    registry = 'chniteesh71/cicd-kube-docker'
                    registryCredentials = 'dockerhub'
    }

    stages{
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage ('Build Docker app image') {
          steps {
            script {
              dockerImage = docker.build( registry + ":V$BUILD_NUMBER" , ".")
            }

          }
        }

        stage ('upload image to docker hub') {
          steps {
            script {
              docker.withRegistry('',registryCredentials) {
                dockerImage.push("V$BUILD_NUMBER")
                dockerImage.push('latest')

              }
            }
          }
        }

        stage('remove the unused docker images') {
          steps {
            sh "docker rmi $registry:V$BUILD_NUMBER"
          }
        }
        stage ('Kubernetes deploy') {
          agent { label 'KOPS'}
          steps {
              sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER} --namespace prod"
          }
        }

    }

}

