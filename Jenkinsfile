@Library("shared-jenkins")_
pipeline {

    agent any

    options {
        // Pipeline options
        ansiColor('xterm')
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: "25", artifactNumToKeepStr: "25"))
        disableConcurrentBuilds()
    }

    stages {
        stage('[Docker] Build and push') {
            steps {
                script {
                    utilities.buildAndPushPython('jinja2')
                }
            }
        }
    }
}