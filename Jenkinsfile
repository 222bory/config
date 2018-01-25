pipeline {

    agent any

    stages {
        stage ('CheckOut') {
            checkout scm
        }

        state ('Text') {
            sh './gradlew check'
        }

        stage ('Build') {
            sh './gradlew build'
        }

        stage ('Build') {
            sh './gradlew dockerPush'
        }
    }


}