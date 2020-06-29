#!/usr/bin/env groovy
pipeline {
    agent {
        label 'connector'
    }

    options {
        ansiColor("xterm")
    }

    parameters {
        string(name: 'GIT_CHECKOUT', defaultValue: 'master', description: 'Branch, tag, commit')
        booleanParam(name : 'PUBLISH_BUILD', defaultValue: false, description: 'Publish to maven')
    }

    stages {
        stage('SCM Checkout') {
            steps {
                git(url: 'git@github.com:lensesio/sql-core.git', credentialsId: '21e9ca7e-fdad-40dc-a1a5-7ea4959f1e45', branch: params.GIT_CHECKOUT)
                buildName '#${BUILD_NUMBER}, ' + params.GIT_CHECKOUT
            }
        }
        stage('Build') {
            steps {
                script {
                    docker.image('gradle:5.1-jdk8').inside {
                        sh './gradlew clean build'
                    }
                }
            }
        }
        stage('Publish') {
            when {
                expression {
                    GIT_BRANCH = 'origin/' + sh(returnStdout: true, script: 'git rev-parse --abbrev-ref HEAD').trim()
                    return GIT_BRANCH == 'origin/master' && params.PUBLISH_BUILD
                }
            }
            steps {
                script {
                    docker.image('gradle:5.1-jdk8').inside {
                        withCredentials([
                            file(credentialsId: 'a3a9a2bd-6c2e-40bf-be0f-1e24f3597d90', variable: 'GRADLE_PROPERTIES'),
                            file(credentialsId: 'e5c260e3-0bc6-4e7d-a4c0-4138d6305a8b', variable: 'SIGNING_GPG_KEY')
                        ]) {
                            sh 'cat $GRADLE_PROPERTIES > gradle.properties '
                            sh 'echo -e "\nsigning.secretKeyRingFile=$SIGNING_GPG_KEY" >> gradle.properties'
                            sh './gradlew uploadArchives'
                            sh 'rm -f gradle.properties'
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts('build/libs/*.jar')
        }
    }
}
