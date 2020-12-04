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
                    image 'python:3.5'
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
                  pip install markupsafe setuptools==30.3.0
                  python3 setup.py sdist bdist_wheel
                  chown -R jenkins:jenkins .
                '''
                sh '''pwd; ls;
                '''
                stash includes: 'dist/spotipy*.whl', name: 'wheel_artifacts'
            }
        }

        stage("Upload artifacts to nexus"){
            steps{
                unstash 'wheel_artifacts'
                script {
                    utilities.uploadPython wheel: 'dist/spotipy*.whl'
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
