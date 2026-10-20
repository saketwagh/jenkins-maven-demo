pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 15, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '5'))
    }

    tools {
        maven 'maven3'
    }

    environment {
        APP_VERSION = "1.0.${BUILD_NUMBER}"
    }

    stages {
        stage('Info') {
            steps {
                script {
                    env.GIT_SHORT = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    env.IS_PR = (env.CHANGE_ID != null) ? 'yes' : 'no'
                }
                echo "Branch: ${env.BRANCH_NAME}, pull request: ${env.IS_PR}, version: ${APP_VERSION}-${env.GIT_SHORT}"
            }
        }
        stage('Test') {
            steps {
                sh 'mvn -B -ntp clean test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Package') {
            steps {
                sh "mvn -B -ntp -DskipTests -Drevision=${APP_VERSION}-${env.GIT_SHORT} package"
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
        stage('Pull request checks') {
            when { changeRequest() }
            steps {
                echo "PR #${env.CHANGE_ID}: ${env.CHANGE_BRANCH} into ${env.CHANGE_TARGET}"
            }
        }
        stage('Publish (main only)') {
            when { branch 'main' }
            steps {
                echo "Pretend publish of calc-${APP_VERSION}-${env.GIT_SHORT}.jar to an artifact repository"
            }
        }
    }
}
