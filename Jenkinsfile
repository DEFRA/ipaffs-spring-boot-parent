import defra.pipeline.config.Config
import defra.pipeline.deploy.DeployQueries

def deployables = DeployQueries.getListOfDeployableComponents(Config.getPropertyValue("secretscanningDeploymentList", this), this)

@Library('pipeline-library') _

pipeline {
    agent { label 'autoNodeLive' }
    environment {
        SERVICE_NAME = "spring-boot-parent"
        JAVA_HOME = "/usr/lib/jvm/temurin-11-jdk-amd64"
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
        stage('Code Secret Scanning') {
            when {
                expression { deployables.contains(SERVICE_NAME) }
            }
            steps {
                build job: 'Code-secret-scanning',
                        parameters: [
                                string(name: 'repository', value: "https://giteux.azure.defra.cloud/imports/$SERVICE_NAME" + ".git"),
                                string(name: 'branch', value: "master")
                        ]
            }
        }

        stage('===Snapshot=== Update Version') {
            when {
                anyOf {
                    branch 'feature/**'
                    branch 'bugfix/**'
                }
            }
            steps {
                getFile 'settings/maven.xml'
                script {
                    pomVersion = sh script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true
                }
                mvnVersionUpdate("${pomVersion}", "${BRANCH_NAME}", "pom.xml")
            }
        }

        stage('Package') {
            steps {
                getFile 'settings/maven.xml'
                sh 'mvn -f pom.xml resources:resources --settings ./settings/maven.xml'
                sh 'mvn -f pom.xml package --settings ./settings/maven.xml'
            }
        }

        stage('===Snapshot=== Deploy Maven Artifact') {
            when {
                anyOf {
                    branch 'feature/**'
                    branch 'bugfix/**'
                }
            }
            steps {
                mvnSnapshotDeploy("pom.xml")
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
                allOf {
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
            notifyTeams('red', "JOB: ${env.JOB_NAME} BUILD NUMBER: ${env.BUILD_NUMBER}", 'FAILED', 'mergeNotifications')
            updateGitlabCommitStatus name: 'build', state: 'failed'
        }
        success {
            notifyTeams('green', "JOB: ${env.JOB_NAME} BUILD NUMBER: ${env.BUILD_NUMBER}", 'SUCCESS', 'mergeNotifications')
            updateGitlabCommitStatus name: 'build', state: 'success'
        }
    }
}
