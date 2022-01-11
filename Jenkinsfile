@Library('pipeline-library@hotfix/IMTA-123-test-old-hotfix') _

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
        sh 'mvn -f pom.xml resources:resources --settings ./settings/maven.xml'
        sh 'mvn -f pom.xml package --settings ./settings/maven.xml'
      }
    }

    stage('Deploy Maven Artifact') {

      when {
         branch 'hotfix/**'
      }
      steps {
        env.NEXT_VERSION = input message: 'Please enter the next snapshot version',
                                   parameters: [string(defaultValue: '',
                                                description: 'E.g 2.0.103-HOTFIX-2-SNAPSHOT where 2.0.103 is the hotfixed version and -2- is the hotfix version',
                                                name: 'Next snapshot version')]
        mvnDeploy("${BRANCH_NAME}", "pom.xml", env.NEXT_VERSION)
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