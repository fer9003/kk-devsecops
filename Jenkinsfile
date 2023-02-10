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

      stage('Docker Build and Push') {
        steps {
          sh 'printenv'
          sh 'docker build -t f90mora/devsecopsdemo1:""$GIT_COMMIT"" .'
          sh 'docker push f90mora/devsecopsdemo1:""$GIT_COMMIT""'
        }
      }
    }
}
