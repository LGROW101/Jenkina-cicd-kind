pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: golang
                    image: golang:1.21
                    command:
                    - cat
                    tty: true
                  - name: docker
                    image: docker:dind
                    securityContext:
                      privileged: true
                    volumeMounts:
                    - name: docker-socket
                      mountPath: /var/run/docker.sock
                  volumes:
                  - name: docker-socket
                    hostPath:
                      path: /var/run/docker.sock
            '''
        }
    }
    environment {
        DOCKER_IMAGE = 'your-dockerhub-username/assessment-tax:v1'
        DOCKER_CREDENTIALS = credentials('docker-hub-credentials')
    }
    stages {
        stage('Unit Test') {
            steps {
                container('golang') {
                    sh 'go mod download'
                    sh 'go test ./... -v -coverprofile=coverage.out'
                    sh 'go tool cover -func=coverage.out'
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'coverage.out', fingerprint: true
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
    post {
        always {
            cleanWs()
        }
    }
}