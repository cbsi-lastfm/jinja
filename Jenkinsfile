pipeline {

    agent any

    options {
        //Pipeline options
        ansiColor('xterm')
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: "20", artifactNumToKeepStr: "20"))
        skipDefaultCheckout()
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = 'gcr.io/i-lastfm-tools/jinja2'
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
                  #Needed for installing flake8
                  pip install --upgrade pip
                  pip install flake8 flake8-print setuptools==30.3.0
                  python3 setup.py develop
                  py.test
                  flake8 | tee flake8.log
                  chown -R jenkins:jenkins ~/*
                '''
                script {
                    versionNum = sh(returnStdout:true, script:'cd src; python3 -c \'import setup; print(setup.version)\'').trim()
                }
            }


            post{
                always{
                    cobertura coberturaReportFile: 'src/coverage.xml'
                    junit "src/junit.xml"
                    recordIssues(
                        tools: [pep8(pattern:'src/flake8.log')],
                        healthy: 10,
                        unhealthy: 20,
                        unstableTotalAll: 150,
                        sourceDirectory: 'src/'
                    )
                    cleanWs deleteDirs: true
                }
            }
        }

        stage('Build docker image'){
            steps{
                unstash 'sources'
                echo "Building docker image $IMAGE_NAME for ${GIT_COMMIT} with v${versionNum}"
                sh '''
                cd src
                  if [[ "$BRANCH_NAME" == "master" ]]
                  then
                      docker build --pull -t $IMAGE_NAME:master-head .
                      docker push $IMAGE_NAME:master-head
                      docker tag $IMAGE_NAME:master-head $IMAGE_NAME:v${versionNum} || true
                      docker push $IMAGE_NAME:v${versionNum}
                  else
                      docker build --pull -t $IMAGE_NAME:latest .
                      docker push $IMAGE_NAME:latest
                  fi
                '''
            }
        }
    }
}
