pipeline {
    agent any

    tools {
        // Ensure 'Maven3' matches the name configured in Jenkins > Manage Jenkins > Global Tool Configuration
        maven 'Maven3'
        // Ensure 'Java21' matches the name configured for your JDK installation
        jdk 'Java21'
    }

    environment {
        // This variable holds the SonarQube installation name from Jenkins config
        SONARQUBE_ENV = 'MySonarQube'
        // Define Nexus credentials ID for better readability
        // YOU MUST CREATE A JENKINS CREDENTIAL WITH THIS ID (Username with password)
        NEXUS_CREDENTIALS_ID = 'nexus-deploy-credentials'
        // Define Maven settings file ID from Managed Files
        // YOU MUST CREATE A JENKINS MANAGED FILE (Maven settings file) WITH THIS ID
        MAVEN_SETTINGS_FILE_ID = 'my-nexus-settings'
        // This ID must match the actual credential ID in Jenkins (from your screenshot)
        // YOU MUST CREATE A JENKINS CREDENTIAL WITH THIS ID (Username with password)
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
                // Using '-DskipTests' to avoid running tests during the build phase if they aren't critical
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Ensure the variable SONARQUBE_ENV correctly refers to your configured SonarQube installation
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    // It's good practice to specify sonar.projectKey if not in sonar-project.properties or pom.xml
                    sh 'mvn sonar:sonar -Dsonar.projectKey=SimpleCustomerApp'
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                script {
                    // This block securely retrieves your Nexus username and password from Jenkins credentials
                    // and makes them available as environment variables NEXUS_USERNAME and NEXUS_PASSWORD.
                    withCredentials([usernamePassword(credentialsId: NEXUS_CREDENTIALS_ID, passwordVariable: 'NEXUS_PASSWORD', usernameVariable: 'NEXUS_USERNAME')]) {
                        // This block integrates Maven, using the managed settings.xml file
                        // which contains the server definition that uses the NEXUS_USERNAME and NEXUS_PASSWORD.
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
                    // This block securely retrieves your Tomcat username and password from Jenkins credentials
                    // and makes them available as environment variables TOMCAT_USERNAME and TOMCAT_PASSWORD.
                    // This is crucial for Maven to pick up the credentials from settings.xml.
                    withCredentials([usernamePassword(credentialsId: TOMCAT_CREDENTIALS_ID, passwordVariable: 'TOMCAT_PASSWORD', usernameVariable: 'TOMCAT_USERNAME')]) {
                        // This ensures Maven runs with your custom settings.xml (which now contains Tomcat server credentials)
                        withMaven(maven: 'Maven3', mavenSettingsConfig: MAVEN_SETTINGS_FILE_ID) {
                            // This command triggers the Maven plugin to deploy the WAR file to Tomcat.
                            // Make sure your pom.xml is configured with tomcat9-maven-plugin.
                            sh "mvn tomcat7:redeploy"
                        }
                    }
                }
            }
        }
    }

    // This is the ONLY post block for the entire pipeline.
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
