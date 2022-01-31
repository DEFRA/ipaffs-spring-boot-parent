@Library('pipeline-library@feature/IMTA-9847-hotfix-process-for-libraries') _

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
  parameters {
      booleanParam(name: 'PUSH_TO_ARTIFACTORY', defaultValue: false, description: 'Do you want to push hotfix to artifactory?')
      string(name: 'AFFECTED_VERSION', defaultValue: null, description: 'Enter affected version to be hotfixed e.g 2.0.103')
      string(name: 'HOTFIX_VERSION', defaultValue: '1', description: 'Enter hotfix version (1 if it’s the 1st time patching this release version)')
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
        environment name: 'BRANCH_NAME', value: 'master'
      }

      steps {
        mvnDeploy("${BRANCH_NAME}", "pom.xml")
      }
    }

    stage('===HOTFIX=== Deploy Maven Artifact') {
        when {
            allOf{
                branch 'hotfix/**'
                expression { params.PUSH_TO_ARTIFACTORY == true }
                expression { params.AFFECTED_VERSION != null }
            }
        }
        steps {
            mvnDeploy("${BRANCH_NAME}", "pom.xml", true)
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