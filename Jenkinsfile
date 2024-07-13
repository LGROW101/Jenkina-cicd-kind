pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: kubectl
                    image: bitnami/kubectl:latest
                    command:
                    - cat
                    tty: true
            '''
        }
    }
    
    stages {
        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    sh '''
                        kubectl set image deployment/assessment-tax-api assessment-tax-api=locker01/assessment-tax-api:v1
                        kubectl rollout status deployment/assessment-tax-api
                    '''
                }
            }
        }
    }
}