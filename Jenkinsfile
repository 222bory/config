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
        echo "Repository : ${repository}/${reference}"
        echo "${env.BUILD_URL}"
		echo "${after} => ${before}"
    }

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
                body: "Please go to ${env.BUILD_URL}."
        }
    }

    stage('Deploy check') {
        mail subject: "Jenkins Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) is waiting for input",
            to: 'blue.park@kt.com',
            body: "Please go to ${env.BUILD_URL}."

        def userInput = true
        def didTimeout = false

        try {
            timeout(time: 1, unit: 'MINUTES') {
                userInput = input(id: 'Proceed1', message: '배포하시겠습니까?', parameters: [
                    [$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']
                    ])
            }
        } catch(err) {
            def user = err.getCauses()[0].getUser()
            if('SYSTEM' == user.toString()) { // SYSTEM means timeout.
                didTimeout = true
            } else {
                userInput = false
                echo "Aborted by: [${user}]"
            }
        }

        if (didTimeout) {
            // do something on timeout
            echo "no input was received before timeout"
             currentBuild.result == 'SUCCESS'
        } else if (userInput == true) {
            currentBuild.result == 'SUCCESS'
            echo "this was successful"
        } else {
            currentBuild.result = 'FAILURE'
        }
    }

    stage('Deploy') {
        echo 'Build Result : ' + currentBuild.result

        if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
            sh 'true'
        }
    }
}