pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: golang
                    image: golang:1.22.1
                    command:
                    - cat
                    tty: true
                  - name: docker
                    image: docker:dind
                    securityContext:
                      privileged: true
                    env:
                    - name: DOCKER_TLS_CERTDIR
                      value: ""
                    command:
                    - dockerd-entrypoint.sh
                    tty: true
            '''
        }
    }
    environment {
        DOCKER_IMAGE = 'locker01/assessment-tax:v1'
        DOCKER_CREDENTIALS = credentials('docker-hub-credentials')
    }
    stages {
        stage('Unit Test') {
            steps {
                container('golang') {
                    sh 'go version'
                    sh 'go mod download'
                    sh 'go test ./... -v -coverprofile=coverage.out'
                    sh 'go tool cover -func=coverage.out'
                    sh 'go tool cover -html=coverage.out -o coverage.html'
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'coverage.out,coverage.html', fingerprint: true
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: '.',
                        reportFiles: 'coverage.html',
                        reportName: 'Go Coverage Report'
                    ])
                }
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                container('docker') {
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh 'docker push ${DOCKER_IMAGE}'
                    }
                }
            }
        }
    }
}