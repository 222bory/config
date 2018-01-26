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
		echo "${after} => ${before}"
    }

    stage('Checkout') {
        checkout scm
    }

    stage('Test') {
        sh './gradlew check || true'
    }

    stage('Build') {
        sh './gradlew build dockerPush'
        archiveArtifacts artifacts: '**/build/libs/*.jar', fingerprint: true
    }

    stage('Deploy check') {
        mail (to: 'blue.park@kt.com',
            subject: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) is waiting for input",
            body: "Please go to ${env.BUILD_URL}.");

        def userInput = true
        def didTimeout = false

        try {
            timeout(time: 1, unit: 'MINUTES') {
                userInput = input(id: 'Proceed1', message: '배포하시겠습니까?', parameters: [
                    [$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']
                    ])
            }
        } catch(err) { // timeout reached or input false
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
            currentBuild.result = 'FAILURE'
        } else if (userInput == true) {
            // do something
            echo "this was successful"
        } else {
            // do something else
            echo "this was not successful"
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