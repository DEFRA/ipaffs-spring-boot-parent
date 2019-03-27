@Library('pipeline-library@feature/IMTA-4699-disable-repocentral-mirror') _

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
        sh 'mvn -f spring-boot-parent/pom.xml package'
        sh 'mvn -f integration-test-parent/pom.xml package'
      }
    }

    stage('Deploy Maven Artifact') {

      when {
        environment name: 'BRANCH_NAME', value: 'feature/IMTA-4605-no-mulitmodule-parent-poms'
      }

      steps {
        mvnDeploy("${BRANCH_NAME}", "spring-boot-parent/pom.xml")
        mvnDeploy("${BRANCH_NAME}", "integration-test-parent/pom.xml")
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