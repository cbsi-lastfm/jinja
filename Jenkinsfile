@Library("shared-jenkins")_
pipeline {

    agent any

    options {
        //Pipeline options
        ansiColor('xterm')
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: "25", artifactNumToKeepStr: "25"))
        disableConcurrentBuilds()
    }
    stages {

        stage('Clean Workspace') {

            steps {
                cleanWs()
                dir('src') {
                    checkout scm
                }
                stash name: 'sources'
                script {
                    GIT_COMMIT = sh(returnStdout:true, script:'cd src; git rev-parse HEAD').trim()
                }
            }
        }

        stage("Run unit tests") {

            agent {
                docker {
                    image 'python:3.5'
                    args '-u root:root -v /etc/passwd:/etc/passwd -v /etc/group:/etc/group'
                }
            }

            steps {
                unstash 'sources'
                echo "Building ${GIT_COMMIT}"
                sh '''
                  export HOME=$WORKSPACE
                  export PATH=$HOME/.local/bin:$PATH
                  cd src
                  pip install --upgrade pip
                  pip install markupsafe setuptools==30.3.0
                  python3 setup.py sdist bdist_wheel
                  chown -R jenkins:jenkins .
                '''
                stash includes: 'Jinja2*.egg', name: 'egg_artifacts'
                stash includes: 'Jinja2*.whl', name: 'wheel_artifacts'
            }
        }

        stage("Upload artifacts to nexus"){
            steps{
                unstash 'egg_artifacts'
                unstash 'wheel_artifacts'
                script {
                    utilities.uploadPython egg: '*.egg', wheel: '*.whl'
                }
            }
        }
    }

    post {
        always {
            cleanWs cleanWhenFailure: false
        }
    }
}
