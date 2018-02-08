#!/usr/bin/env groovy
node {
	properties([
        pipelineTriggers([
            [$class: 'GenericTrigger',
                genericVariables: [
                    [expressionType: 'JSONPath', key: 'reference', value: '$.ref'],
                    [expressionType: 'JSONPath', key: 'before', value: '$.before'],
                    [expressionType: 'JSONPath', key: 'after', value: '$.after'],
                    [expressionType: 'JSONPath', key: 'repository', value: '$.repository.full_name']
                ],
                genericRequestVariables: [
                    [key: 'requestWithNumber', regexpFilter: '[^0-9]'],
                    [key: 'requestWithString', regexpFilter: '']
                ],
                genericHeaderVariables: [
                    [key: 'headerWithNumber', regexpFilter: '[^0-9]'],
                    [key: 'headerWithString', regexpFilter: '']
                ],
                regexpFilterText: '$repository/$reference',
                regexpFilterExpression: 'MSA/config/refs/heads/master'
            ]
        ])
    ])

    stage('Checkout') {
        checkout scm
    }

    stage('Test') {
        sh './gradlew check || true'
    }

    stage('Build') {
        try {
            sh './gradlew build dockerPush'
            archiveArtifacts artifacts: '**/build/libs/*.jar', fingerprint: true
        } catch(e) {
            mail subject: "Jenkins Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) failed with ${e.message}",
                to: 'blue.park@kt.com',
                body: "Please go to $env.JENKINS_URL."
        }
    }

    stage('Deploy') {
        sh 'kubectl apply --namespace=development -f deployment.yaml'
    }
}