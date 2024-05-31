pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: gradle
spec:
  containers:
  - name: gradle
    image: gradle:6.3.0-jdk11
    command:
    - cat
    tty: true
  restartPolicy: Never
"""
    }
  }
  environment {
    SCP_CREDS = credentials('scpCredentials')
    DOCKER_REGISTRY = "<< SCR Private Endpoint URI >>"
  }
  stages {
    stage('Approval') {
      when {
        branch 'main'
      }
      steps {
        script {
          def plan = "${env.JOB_NAME} CI!!"
          input message: "Do you want to build and push?",
              parameters: [text(name: 'Plan', description: 'Please review the work', defaultValue: plan)]
        }
      }
    }
    stage('build and push docker image') {
      when {
        branch 'main'
      }
      steps {
        container('gradle') {
          sh "gradle jib --no-daemon --image ${DOCKER_REGISTRY}/eshop-cartservice:latest -Djib.to.auth.username=${SCP_CREDS_USR} -Djib.to.auth.password=${SCP_CREDS_PSW}"
        }
      }
      post {
        success {
          slackSend(channel: '<< slack채널 ID >>', color: 'good', message: "(Job:${env.JOB_NAME}-Build Number : ${env.BUILD_NUMBER}) CI success from <@<< 멤버ID >>>")
        }
        failure {
          slackSend(channel: '<< slack채널 ID >>', color: 'danger', message: "(Job:${env.JOB_NAME}-Build Number : ${env.BUILD_NUMBER}) CI fail from <@<< 멤버ID >>>")
        }
      }
    }
  }
}

