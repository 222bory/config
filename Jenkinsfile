#!/usr/bin/env groovy
node {
	properties([
        pipelineTriggers([
            [$class: 'GenericTrigger',
                genericVariables: [
                    [expressionType: 'JSONPath', key: 'reference', value: '$.ref'],
                    [expressionType: 'JSONPath', key: 'before', value: '$.before']
                ],
                genericRequestVariables: [
                    [key: 'requestWithNumber', regexpFilter: '[^0-9]'],
                    [key: 'repository', regexpFilter: 'config']
                ],
                genericHeaderVariables: [
                    [key: 'headerWithNumber', regexpFilter: '[^0-9]'],
                    [key: 'headerWithString', regexpFilter: '']
                ],
                regexpFilterText: '',
                regexpFilterExpression: ''
            ]
        ])
    ])

    stage("Info") {
        sh '''
        echo Variables from shell:
        echo reference $reference
        echo before $before
        echo Repository $repository
        echo requestWithString $requestWithString
        '''
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