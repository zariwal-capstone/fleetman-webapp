pipeline {
   agent any

   environment {
     // You must set the following environment variables
     // ORGANIZATION_NAME
     // YOUR_DOCKERHUB_USERNAME (it doesn't matter if you don't have one)
     REGISTRY = "zariwal/capstone"
     SERVICE_NAME = "fleetman-webapp"
     REPOSITORY_TAG = "${YOUR_DOCKERHUB_USERNAME}/${ORGANIZATION_NAME}-${SERVICE_NAME}:${BUILD_ID}"
     DOCKERHUB = 'DockerHub'
     PROJECT_NAME = 'fleetman-webapp'
   }

   stages {
      stage('Preparation') {
         steps {
            cleanWs()
            git credentialsId: 'GitHub', url: "https://github.com/${ORGANIZATION_NAME}/${SERVICE_NAME}"
         }
      }

      stage ('OWASP Dependency-Check Vulnerabilities') {
            steps {
                dependencyCheck additionalArguments: '''
                    -o "./"
                    -s "./"
                    -f "ALL"
                    --prettyPrint''', odcInstallation: 'Default'

                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
      }

      stage('Sonarqube Analysis') {
          environment {
              scannerHome = tool 'SonarScanner'
          }
          steps {
              withSonarQubeEnv('SonarQube') {
                  sh "${scannerHome}/bin/sonar-scanner -Dsonar.userHome=`pwd`/.sonar -Dsonar.projectKey=$PROJECT_NAME -X"
              }
/*               timeout(time: 10, unit: 'MINUTES') {
                  waitForQualityGate abortPipeline: true
              } */
          }
      }
/*       stage("Quality gate") {
            steps {
                waitForQualityGate abortPipeline: true
            }
      } */

      stage('Build') {
         steps {
            sh 'echo No build required for Webapp.'
         }
      }

      stage('Build and Push Image') {
         steps {
           sh 'docker image build -t ${REPOSITORY_TAG} .'
         }
      }

      stage('Building our docker image') {
         steps {
             script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
             }
         }
      }

      stage('Push our image dockerhub') {
            steps {
                script {
                    docker.withRegistry( '', DOCKERHUB ) {
                        dockerImage.push()
                    }
                }
            }
      }

      stage('Deploy to Cluster') {
          steps {
            sh 'envsubst < ${WORKSPACE}/deploy.yaml | kubectl apply -f -'
          }
      }
	  stage ("Dynamic Analysis - DAST with OWASP ZAP") {
			steps {
				sh "docker run --network host -t owasp/zap2docker-stable zap-baseline.py -t http://127.0.0.1:50071/ || true"
			}

	  }
   }
}
