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

        stage("Build Project") {

            agent {
                docker {
                    image 'python:3.7'
                    args '-u root:sudo -v /etc/passwd:/etc/passwd -v /etc/group:/etc/group'
                }
            }

            steps {
                unstash 'sources'
                echo "Building ${GIT_COMMIT}"
                sh '''
                  export HOME=$WORKSPACE
                  export PATH=$HOME/.local/bin:$PATH
                  pip install --upgrade pip
                  pip install --upgrade build
                  pip install -r requirements/build.txt
                  python -m pip install .
                  python -m build
                '''
                sh '''pwd; ls; ls dist;
                '''
                stash(name: 'wheel_artifacts', includes: '*.whl')
            }
        }

        stage("Upload artifacts to nexus"){
            steps{
                unstash 'wheel_artifacts'
                script {
                    utilities.uploadPython wheel: 'Jinja2*.whl'
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
