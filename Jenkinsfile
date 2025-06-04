pipeline {
  agent any

  tools {
    maven 'Maven3'
    jdk 'Java17'
  }

  environment {
    SONARQUBE_ENV = 'MySonarQube'
  }

  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/betawins/sabear_simplecutomerapp.git', branch: 'main'
      }
    }

    stage('Build') {
      steps {
        sh 'mvn clean package'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv("${SONARQUBE_ENV}") {
          sh 'mvn sonar:sonar'
        }
      }
    }

    stage('Upload to Nexus') {
      steps {
        sh 'mvn deploy'
      }
    }

    stage('Deploy to Tomcat') {
      steps {
        sh '''
        curl -u tomcat:yourpassword --upload-file target/SimpleCustomerApp.war \
        "http://3.86.140.185:8080/manager/text/deploy?path=/SimpleCustomerApp&update=true"
        '''
      }
    }
  }
}
