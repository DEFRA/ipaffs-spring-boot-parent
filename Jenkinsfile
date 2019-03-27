@Library('pipeline-library') _

pipeline {
  agent {label 'swarm'}
  environment {
    SERVICE_NAME = "spring-boot-parent"
  }
  options {
    ansiColor('xterm')
    timestamps()
    disableConcurrentBuilds()
  }

  stages {

    stage('Package') {
      steps {
        getFile 'settings/maven.xml'
        sh 'mvn -f pom.xml package'
      }
    }

    stage('Deploy Maven Artifact') {

      when {
        environment name: 'BRANCH_NAME', value: 'master'
      }

      steps {
        mvnDeploy("${BRANCH_NAME}", "pom.xml")
      }
    }
  }

  post {
      always {
        step([$class: 'WsCleanup'])
      }
      failure {
        notifySlack("BUILD OF ${SERVICE_NAME} HAS FAILED: ${BUILD_URL}", "#FF9FA1")
      }
  }
}