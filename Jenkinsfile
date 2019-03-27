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
        sh 'mvn -f spring-boot-parent/pom.xml package'
        sh 'mvn -f integration-test-parent/pom.xml package'
      }
    }

    stage('Deploy Maven Artifact') {

      when {
        environment name: 'BRANCH_NAME', value: 'master'
      }

      steps {
        script {
          // TODO: would be nice if this was a conditional step instead
          result = sh(script: "git log -1 | grep '\\[maven-release-plugin\\]'", returnStatus: true)
          if(result != 0) {
              echo "Running mvn release..."
          } else {
              echo "SKIPPING mvn release: the previous commit was a release commit"
              return
          }

          String mvnReleaseGitUserEmail = Config.getPropertyValue('mvnReleaseGitUserEmail', this)
          String mvnReleaseGitUserName = Config.getPropertyValue('mvnReleaseGitUserName', this)

          // Jenkins by default checks out a detached head (no local branch name)
          // The maven release plugin fails without a local branch name
          sh(script: "git checkout ${BRANCH_NAME}")
          sh(script: "git config user.email \"${mvnReleaseGitUserEmail}\"")
          sh(script: "git config user.name \"${mvnReleaseGitUserName}\"")
          withCredentials([
                  usernamePassword(credentialsId: 'artifactoryImportsCreds', passwordVariable: 'RELEASE_PASSWORD', usernameVariable: 'USERNAME'),
                  string(credentialsId: 'JENKINS_GITLAB_TOKEN', variable: 'GITLAB_TOKEN')]) {
              sh(script: "mvn -f spring-boot-parent/pom.xml -B release:prepare release:perform --settings ./settings/maven.xml -DskipTests=true")
              sh(script: "mvn -f spring-boot-parent/pom.xml -B release:prepare release:perform --settings ./settings/maven.xml -DskipTests=true")
          }
        }
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