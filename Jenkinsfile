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

        userAborted = false
        startMillis = System.currentTimeMillis()
        timeoutMillis = 10000

        try {
            timeout(time: 1, unit: 'MINUTES') {
                input "배포하시겠습니까?"
            }
        } catch (org.jenkinsci.plugins.workflow.steps.FlowInterruptedException e) {
            cause = e.causes.get(0)
            echo "Aborted by " + cause.getUser().toString()
            if (cause.getUser().toString() != 'SYSTEM') {
                startMillis = System.currentTimeMillis()
            } else {
            endMillis = System.currentTimeMillis()
                if (endMillis - startMillis >= timeoutMillis) {
                  echo "Approval timed out. Continuing with deployment."
                } else {
                  userAborted = true
                  echo "SYSTEM aborted, but looks like timeout period didn't complete. Aborting."
                }
            }
        }

        if (userAborted) {
          currentBuild.result = 'ABORTED'
        } else {
          currentBuild.result = 'SUCCESS'
          echo "Firing the missiles!"
        }
    }

    stage('Deploy') {
        echo 'Build Result : ' + currentBuild.result

        if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
            sh 'true'
        }
    }
}