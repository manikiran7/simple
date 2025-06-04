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
        // ADDED SLACK CREDENTIALS ID HERE for clarity
        SLACK_CREDENTIALS_ID = 'slack-token' // Your credential ID
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
                        withMaven(maven: 'Maven3', mavenSettingsConfig: MAVEN_SETTINGS_FILE_ID) {
                            sh "mvn tomcat7:redeploy"
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
            echo 'Pipeline cleanup complete.'
            slackSend (
                channel: '#jenkins-builds', // Replace with your actual Slack channel name
                color: '#CCCC00',
                message: "Project *${env.JOB_NAME}* - Build #${env.BUILD_NUMBER} has finished with status: *${currentBuild.currentResult}* (<${env.BUILD_URL}|Open in Jenkins>)"
            )
        }
        success {
            echo 'Pipeline finished successfully!'
            slackSend (
                channel: '#jenkins-builds', // Replace with your actual Slack channel name
                color: 'good',
                message: "SUCCESS: Project *${env.JOB_NAME}* - Build #${env.BUILD_NUMBER} deployed successfully! (<${env.BUILD_URL}|Open in Jenkins>)"
            )
        }
        failure {
            echo 'Pipeline failed!'
            slackSend (
                channel: '#jenkins-builds', // Replace with your actual Slack channel name
                color: 'danger',
                message: "FAILURE: Project *${env.JOB_NAME}* - Build #${env.BUILD_NUMBER} failed! (<${env.BUILD_URL}|Open in Jenkins>)"
            )
        }
        unstable {
            slackSend (
                channel: '#jenkins-builds', // Replace with your actual Slack channel name
                color: 'warning',
                message: "UNSTABLE: Project *${env.JOB_NAME}* - Build #${env.BUILD_NUMBER} is unstable! (<${env.BUILD_URL}|Open in Jenkins>)"
            )
        }
    }
}
