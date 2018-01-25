#!/usr/bin/env groovy

node {

    checkout scm

    state('Test') {
        sh './gradlew check || true'
    }

    stage('Build') {
        sh './gradlew build dockerPush'
        archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
    }

    stage('Deploy') {
        if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
            //sh 'make publish'
        }
    }
}