node {

    stage ('CheckOut') {
        checkout scm
    }

    stage ('Build & Push') {
        sh './gradlew clean build dockerPush'
    }

}