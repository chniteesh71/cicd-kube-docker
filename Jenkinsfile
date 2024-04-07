def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]
pipeline {
     agent any
     tools {
          maven "MAVEN3"
          jdk "OracleJDK8"
     }
     environment {
                    SONARSERVER = 'sonarserver'
                    SONARSCANNER = 'sonarscanner'
                    registry = 'chniteesh71/cicd-kube-docker'
                    registryCredentials = 'dockerhub'


     }
     stages {
          stage ('build') {
               steps {
                    sh 'mvn -s settings.xml -DskipTests install '
               }
               post {
                    success {
                         echo "Now archiving.."
                         archiveArtifacts artifacts: '**/*.war'
                    }
               }
          }

          stage ('test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
          }

          stage ('Checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
          }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
               withSonarQubeEnv("${SONARSERVER}") {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
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
    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
    
}