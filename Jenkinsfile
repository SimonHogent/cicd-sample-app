pipeline {
    agent any
    parameters {
        booleanParam(name: 'AUTO_APPROVE', defaultValue: true, description: 'Skip the manual approval before production')
    }
    stages {
        stage('Staging') {
            steps {
                sh 'docker compose down || true'
                sh 'docker compose up -d --build'
                sh 'docker exec -i todoappdb mariadb -utodo_usr -pletmeinplz todo_db < TodoApp/schema.sql'
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
                        docker compose down || true
                        docker compose up -d --build
                        docker exec -i todoappdb mariadb -utodo_usr -pletmeinplz todo_db < TodoApp/schema.sql
                    '''
                }
            }
        }
    }
}