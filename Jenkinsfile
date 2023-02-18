pipeline {
  agent any

  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "f90mora/devsecopsdemo1:${GIT_COMMIT}"
    applicationURL = "http://localhost"
    applicationURI = "/increment/99"
  }

  //trigger pipeline using webhook
  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }

      stage('Unit Tests - JUnit and Jacoco') {
        steps {
          sh "mvn test"
        } 
      }

      stage('Mutation Test - PIT') {
        steps {
          sh "mvn org.pitest:pitest-maven:mutationCoverage"
        }
        post {
          always {
            pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
          }
        }
      }

      stage('SonarQube- SAST') {
        steps {
          withSonarQubeEnv('SonarQube') {
            sh '''
              mvn clean verify sonar:sonar \
              -Dsonar.projectKey=numeric-application \
              -Dsonar.host.url=http://localhost:9000 \
          '''
          }
          timeout(time:2, unit: 'MINUTES') {
            script {
              waitForQualityGate abortPipeline: true
            }
          }
        }
      }

      stage('Vulnerability Scan - Docker') {
        steps {
          parallel (
            //"Dependency Scan": {
            //  sh "mvn dependency-check:check"
            //},
            "Trivy Scan": {
              sh "bash trivy-docker-image-scan.sh"
            },
            "OPA Conftest": {
              sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
            }
          )
        }
      }

      stage('Docker Build and Push') {
        steps {
          withDockerRegistry([credentialsId: "docker-hub", url:""]) {
          sh 'printenv'
          sh 'sudo docker build -t f90mora/devsecopsdemo1:""$GIT_COMMIT"" .'
          sh 'docker push f90mora/devsecopsdemo1:""$GIT_COMMIT""'
          }
        }
      }


      stage('Vulnerability Scan- Kubernetes') {
        steps {
          parallel(
            "OPA Scan" : {
              sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
            },
            "Kubesec Scan": {
              sh "bash kubesec-scan.sh"
            },
            //"Trivy Scan": {
             // sh "bash trivy-k8s-scan.sh"
            //}
          )
        }
      }


      stage('k8s Deployment - DEV') {
        steps {
          parallel(
            "Deployment" : {
              withKubeConfig([credentialsId: 'kubeconfig']) {
                sh 'bash k8s-deployment.sh'
              }
            },
            "Rollout Status": {
              withKubeConfig([credentialsId: 'kubeconfig']) {
                sh 'bash k8s-deployment-rollout-status.sh'
              }
            }
          )
        }
      }

    }

    post {
      always {
         junit 'target/surefire-reports/*.xml'
         jacoco execPattern: 'target/jacoco.exec'
         dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
      }
    }
}
