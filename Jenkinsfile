pipeline {
    agent any
    parameters {
        booleanParam(name: 'AUTO_APPROVE', defaultValue: true, description: 'Skip the manual approval before production')
    }
    stages {
        stage('Staging') {
            steps {
                sh 'docker rm -f samplerunning || true'
                sh 'rm -rf tempdir'
                sh 'bash ./sample-app.sh'
            }
        }
        stage('Test staging') {
            steps {
                sh '''
                    sleep 3
                    IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' samplerunning)
                    curl -s http://$IP:5050/ | grep "You are calling me from"
                '''
            }
        }
        stage('Approve production') {
            when { expression { params.AUTO_APPROVE == false } }
            steps {
                input message: 'Deploy to production (dmz010)?'
            }
        }
        stage('Production') {
            steps {
                sshagent(['dmz010-deploy']) {
                    sh '''
                        mkdir -p ~/.ssh
                        ssh-keyscan -H 192.0.2.10 >> ~/.ssh/known_hosts
                        export DOCKER_HOST=ssh://simon@192.0.2.10
                        docker rm -f samplerunning || true
                        rm -rf tempdir
                        BIND_ADDR=127.0.0.1 bash ./sample-app.sh
                    '''
                }
            }
        }
    }
}