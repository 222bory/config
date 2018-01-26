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
            def filter = 'all'

            waitUntil {
                def bRun = build job: 'jobX', propagate: false  // To avoid failfast/propagate errors
                if (bRun.getRawBuild().result.isWorseOrEqualTo(hudson.model.Result.FAILURE)) {
                    def userInput = true
                    def didTimeout = false
                    def inputValue

                    try {

                        // Timeout in case to avoid running this forever
                        timeout(time: 60, unit: 'SECONDS') {
                            inputValue = input(
                                id: 'userInput', message: 'jobX failed let\'s rerun it?', parameters: [
                                    [$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']
                                ])
                        }
                    } catch(err) { // timeout reached or input false
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
                        echo "no input was received before timeout"
                        false // false will cause infinite loops but it's a way to keep retrying, otherwise use true and will exit the loop
                    } else if (userInput == true) {
                        echo "this was successful"
                        false // if user answered then iterate through the loop again
                    } else {
                        echo "this was not successful"
                        currentBuild.result = 'ABORTED'
                        true  // if user aborted the input then exit the loop
                    }
                } else {
                    currentBuild.result = 'SUCCESS'
                    true  // if build finished successfully as expected then exit the loop
                }
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