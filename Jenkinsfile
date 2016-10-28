pipeline {
    agent label: ''

    tools {
        jdk 'JDK8_66'
    }

    environment {
        SLACK_CHANNEL = "#ci"
        DOCKER_FOLDER = "target/docker"
    }

    stages {
        stage('checkout') {
            checkout scm
        }
        stage('build artifact') {
            sh './mvnw -V clean package'
        }
        stage('build image') {
            sh "mkdir -p target/docker"
            sh "cp src/main/docker/Dockerfile target/docker/"
            sh "cp target/javaee8-mvc.war target/docker/"

            dir("$DOCKER_FOLDER") {
                script {
                    docker.build("eddumelendez/mvc:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('publish image') {
            input message: 'Publish?', submitter: 'edsonchavez'

            dir("$DOCKER_FOLDER") {
                script {
                    def image = docker.build("eddumelendez/mvc:${env.BUILD_NUMBER}")
                    image.push()
                }
            }
        }
    }

    postBuild {
      success {
        archiveArtifacts artifacts: 'target/*.war'
      }
    }

    notifications {
        always {
            echo "I HAVE FINISHED"
        }
        success {
            slackSend message: "Job ${env.JOB_NAME} Build ${env.BUILD_NUMBER}", channel: "$SLACK_CHANNEL", color: "good"
        }
        failure {
            slackSend message: "Job ${env.JOB_NAME} Build ${env.BUILD_NUMBER}", channel: "$SLACK_CHANNEL", color: "danger"
        }
        unstable {
            slackSend message: "Job ${env.JOB_NAME} Build ${env.BUILD_NUMBER}", channel: "$SLACK_CHANNEL", color: "warning"
        }
        changed {
            slackSend message: "Job ${env.JOB_NAME} Build ${env.BUILD_NUMBER}", channel: "$SLACK_CHANNEL", color: "warning"
        }
    }
}
