pipeline {
  agent any
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
        post {
          always {
            junit 'target/surefire-reports/*.xml'
            jacoco execPattern: 'target/jacoco.exec'
          }
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
          sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-applications -Dsonar.host.url=http://devsecops-demo-fm.eastus.cloudapp azure.com:9000 -Dsonar.login=sqp_a229d06054e975052d892b2d792963f7df6faeac"
        }
      }

      stage('Docker Build and Push') {
        steps {
          withDockerRegistry([credentialsId: "docker-hub", url:""]) {
          sh 'printenv'
          sh 'docker build -t f90mora/devsecopsdemo1:""$GIT_COMMIT"" .'
          sh 'docker push f90mora/devsecopsdemo1:""$GIT_COMMIT""'
          }
        }
      }

      stage('Kubernetes Deployment - DEV') {
        steps {
          withKubeConfig([credentialsId: "kubeconfig"]) {
            sh "sed -i 's#replace#f90mora/devsecopsdemo1:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
            sh "kubectl apply -f k8s_deployment_service.yaml"
          }
        }
      }
    }
}
