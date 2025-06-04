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
                        sh '''
                        curl -u ${TOMCAT_USERNAME}:${TOMCAT_PASSWORD} --upload-file target/SimpleCustomerApp.war \
                        "http://3.88.144.100:8080/manager/text/deploy?path=/SimpleCustomerApp&update=true"
                        '''
                    }
                }
            }
        }
    }

    post {
        // Move cleanWs() out of 'always' if you want it after all stages.
        // A common practice is to have it in 'always' for cleanup, but place it
        // within the `post` block, after the last stage.
        // Alternatively, use 'success' or 'failure' if you want conditional cleanup.

        // A robust way to ensure cleanup after *all* stages, including post-build steps:
        // No, this is incorrect. The 'always' block is at the *pipeline* level.
        // The problem is that the `sh` step for deploy is within a `script` block,
        // which might behave differently or `cleanWs` was inadvertently clearing early.

        // Let's explicitly put cleanWs() *outside* the individual stage's 'post' actions,
        // and in the *pipeline's* `post` section. This is what you had, but it seems
        // to have executed before the 'Deploy to Tomcat' stage. This is unusual
        // for declarative pipelines.

        // Let's try placing cleanWs() within the 'success' and 'failure' blocks for clarity
        // or ensure it's at the very end.

        // Re-reading your log, the `cleanWs` occurs *after* the `Deploy to Tomcat` stage is skipped.
        // The *reason* `Deploy to Tomcat` is skipped is due to "earlier failure(s)".
        // And the error is "curl: Can't open 'target/SimpleCustomerApp.war'".
        // This implies the WAR file was not created, or the `Build` stage might have failed
        // to produce it, and then SonarQube / Nexus stages just ran without it.

        // Let's check the build stage again.
        // [INFO] Packaging webapp
        // [INFO] Assembling webapp [SimpleCustomerApp] in [/var/lib/jenkins/workspace/first-2/target/SimpleCustomerApp-1.0-SNAPSHOT]
        // [INFO] Processing war project
        // [INFO] Building war: /var/lib/jenkins/workspace/first-2/target/SimpleCustomerApp-1.0-SNAPSHOT.war
        // This confirms the WAR *was* built in the 'Build' stage.

        // The sequence in your log:
        // [Pipeline] // stage (Upload to Nexus)
        // [Pipeline] stage { (Deploy to Tomcat)
        // Stage "Deploy to Tomcat" skipped due to earlier failure(s)
        // [Pipeline] getContext
        // [Pipeline] }
        // [Pipeline] // stage
        // [Pipeline] stage { (Declarative: Post Actions)
        // [Pipeline] cleanWs

        // This means the pipeline failed *before* the 'Deploy to Tomcat' stage was entered for execution,
        // and then the 'Deploy to Tomcat' stage was SKIPPED, and then 'cleanWs' ran.
        // The error `curl: Can't open 'target/SimpleCustomerApp.war'` came from a *previous* run's log or was mixed in.
        // Apologies for the misinterpretation of the `cleanWs` timing based on my immediate reaction.

        // THE ACTUAL PROBLEM: The "Deploy to Tomcat" stage is SKIPPED.
        // Why was it skipped? "due to earlier failure(s)".
        // What was the earlier failure? The `Upload to Nexus` stage.
        // But your current log snippet *shows* 'Upload to Nexus' as BUILD SUCCESS.
        // This is contradictory.

        // Let's re-examine the full log of the *latest* run.
        // The log you provided *starts* with "INFO 10:03:17.499 More about the report processing...".
        // It shows "SonarQube Analysis" -> "BUILD SUCCESS"
        // Then "Upload to Nexus" -> "BUILD SUCCESS"

        // The curl error is happening *inside* the `Deploy to Tomcat` stage itself.
        // And the `Deploy to Tomcat` stage is *not* being skipped in this last log.
        // My apologies, I was confused by the *previous* log snippets where it was skipped.
        // In the LATEST log, it is *not* skipped.

        // Okay, so the `curl: Can't open 'target/SimpleCustomerApp.war'` is indeed the problem.
        // This implies the workspace was cleaned *before* the `Deploy to Tomcat` stage.
        // This is *highly unusual* for Declarative Pipeline `post` blocks.

        // Let's confirm if `cleanWs()` was somehow inside the `withMaven` or `withCredentials` block,
        // or if there's an explicit `ws()` block with `cleanWs()` early.

        // Looking at your previous full Jenkinsfile, `cleanWs()` is indeed in the pipeline-level `post/always` block.
        // That means it should run *after* all stages have attempted to execute.

        // The error "curl: Can't open 'target/SimpleCustomerApp.war'" can also mean:
        // 1. The file was indeed deleted (unlikely given `cleanWs` placement)
        // 2. The working directory for `curl` is not `/var/lib/jenkins/workspace/first-2`
        // 3. Permissions issue on the file (less likely if Jenkins built it)

        // Let's add a `pwd` and `ls -l target/` right before the curl command to debug.

        stage('Deploy to Tomcat') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: TOMCAT_CREDENTIALS_ID, passwordVariable: 'TOMCAT_PASSWORD', usernameVariable: 'TOMCAT_USERNAME')]) {
                        sh '''
                        echo "Current working directory:"
                        pwd
                        echo "Contents of target/ directory:"
                        ls -l target/
                        
                        curl -u ${TOMCAT_USERNAME}:${TOMCAT_PASSWORD} --upload-file target/SimpleCustomerApp.war \
                        "http://3.88.144.100:8080/manager/text/deploy?path=/SimpleCustomerApp&update=true"
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            // Keep cleanWs here, as it's the standard place for final cleanup.
            // If the WAR file is still not found after adding pwd/ls,
            // then there's a deeper issue with the workspace or how curl is being invoked.
            cleanWs()
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
