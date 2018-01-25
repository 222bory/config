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
         [key: 'requestWithString', regexpFilter: '']
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

    checkout scm

    stage('Test') {
        sh './gradlew check || true'
    }

    stage('Build') {
        sh './gradlew build dockerPush'
        archiveArtifacts artifacts: '**/build/libs/*.jar', fingerprint: true
    }

    stage('Deploy') {
        if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
            //sh 'make publish'
        }
    }
}