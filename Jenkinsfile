pipeline {
    agent any

    stages {
        stage('Atualizando Código') {
            steps {
                git url: 'https://github.com/gui-sousa/nginx-guizin.git', branch: 'master'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    dockerapp = docker.build("guisousa/nginx-guizin:${env.BUILD_ID}", '-f Dockerfile .')
                }
            }
        }

        stage('Docker Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-gsousa') {
                        dockerapp.push('latest')
                        dockerapp.push("{$env.BUILD_ID}")
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                currentBuild.result = 'SUCCESS'
            }
        }

        success {
            echo 'Pipeline executada com sucesso!'
        }

        failure {
            echo 'A execução da pipeline falhou.'
        }

        unstable {
            echo 'A execução da pipeline foi instável.'
        }

        aborted {
            echo 'A execução da pipeline foi abortada.'
        }

        notBuilt {
            echo 'Nenhum build foi executado nesta pipeline.'
        }
    }
}