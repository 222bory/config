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

    stage("Info") {
        echo "${after}"
        echo "${before}"
        echo "${reference}"
        echo "${repository}"
    }

    checkout scm

    stage('Test') {
        sh './gradlew check || true'
    }

    stage('Build') {
        sh './gradlew build dockerPush'
        archiveArtifacts artifacts: '**/build/libs/*.jar', fingerprint: true
    }

    stage('Deploy') {
        echo 'Build Result : ' + currentBuild.result

        if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
            sh 'true'
        }
    }
}