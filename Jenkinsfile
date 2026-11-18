pipeline {
    agent any
    parameters {
        booleanParam(name: 'AUTO_APPROVE', defaultValue: true, description: 'Skip the manual approval before production')
    }
    stages {
        stage('Staging') {
            steps {
                sh '''
                    docker rm -f samplerunning || true
                    rm -rf tempdir
                    bash sample-app.sh
                '''
            }
        }
        stage('Approve production') {
            when { expression { params.AUTO_APPROVE == false } }
            steps { input message: 'Deploy to production (dmz010)?' }
        }
        stage('Production') {
            steps {
                sshagent(['dmz010-deploy']) {
                    sh '''
                        mkdir -p ~/.ssh
                        ssh-keyscan -H 192.0.2.10 >> ~/.ssh/known_hosts
                        export DOCKER_HOST=ssh://simon@192.0.2.10
                        export BIND_ADDR=127.0.0.1
                        docker rm -f samplerunning || true
                        rm -rf tempdir
                        bash sample-app.sh
                    '''
                }
            }
        }
    }
}
