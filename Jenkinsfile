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
          sh '''
            mvn clean verify sonar:sonar \
            -Dsonar.projectKey=numeric-application \
            -Dsonar.host.url=http://localhost:9000 \
            -Dsonar.login=sqp_9e699f344dfda28fc704166a5cc0bd1615358add
          '''
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
