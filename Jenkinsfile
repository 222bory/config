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
        echo "$env.BUILD_URL"
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
                body: "Please go to $env.BUILD_URL."
        }
    }

    stage('Deploy check') {
        mail subject: "Jenkins Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) is waiting for input",
            to: 'blue.park@kt.com',
            body: "Please go to $env.BUILD_URL."

        script {
            def userInput = true
            def didTimeout = false
            def inputValue

            try {

                // Timeout in case to avoid running this forever
                timeout(time: 60, unit: 'SECONDS') {
                    inputValue = input(
                        id: 'userInput', message: '계속 진행하시겠습니까?', parameters: [
                            [$class: 'BooleanParameterDefinition',
                                defaultValue: true,
                                description: '', name: '확인'
                            ]
                        ])
                }
            } catch(err) {
                echo err.toString()
                def user = err.getCauses()[0].getUser()
                if('SYSTEM' == user.toString()) { // SYSTEM means timeout.
                    didTimeout = true
                } else {
                    userInput = false
                    echo "Aborted by: [${user}]"
                }
            }

            if (didTimeout) {
                currentBuild.result = 'ABORTED'
                false
            } else if (userInput == true) {
                false
            } else {
                currentBuild.result = 'ABORTED'
                true
            }
        }

    }

    stage('Deploy') {
        echo 'Build Result : ' + currentBuild.result

        if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
            sh 'true'
        }
    }
}