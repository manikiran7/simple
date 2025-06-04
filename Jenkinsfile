pipeline {
    agent any

    tools {
        maven 'Maven3'
        jdk 'Java21'
    }

    environment {
        SONARQUBE_ENV = 'MySonarQube'
        NEXUS_CREDENTIALS_ID = 'nexus-deploy-credentials'
        MAVEN_SETTINGS_FILE_ID = 'my-nexus-settings'
        TOMCAT_CREDENTIALS_ID = 'tomcat-manager-credentials'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/manikiran7/simple.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=SimpleCustomerApp'
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: NEXUS_CREDENTIALS_ID, passwordVariable: 'NEXUS_PASSWORD', usernameVariable: 'NEXUS_USERNAME')]) {
                        withMaven(maven: 'Maven3', mavenSettingsConfig: MAVEN_SETTINGS_FILE_ID) {
                            sh "mvn deploy -DskipTests"
                        }
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: TOMCAT_CREDENTIALS_ID, passwordVariable: 'TOMCAT_PASSWORD', usernameVariable: 'TOMCAT_USERNAME')]) {
                        // Debugging steps: check current directory and file existence
                        sh '''
                        echo "Current working directory: $(pwd)"
                        echo "Contents of target/ directory:"
                        ls -l target/
                        '''
                        
                        // The curl command on a single line
                        sh "curl -u ${TOMCAT_USERNAME}:${TOMCAT_PASSWORD} --upload-file target/SimpleCustomerApp.war \"http://3.88.144.100:8080/manager/text/deploy?path=/SimpleCustomerApp&update=true\""
                    }
                }
            }
        }
    }

    // This is the ONLY post block for the entire pipeline
    post {
        always {
            cleanWs() // Clean up workspace after build
            echo 'Pipeline cleanup complete.'
        }
        success {
            echo 'Pipeline finished successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
