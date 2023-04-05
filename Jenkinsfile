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
                    dockerapp = docker.build("guisousa/nginx-guizin:${env.BUILD_ID}",
                        '-f Dockerfile .')
                }
            }
        }

        stage('Docker Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-gsousa') {
                        dockerapp.push("{$env.BUILD_ID}")
                        dockerapp.push("{$env.BUILD_ID}")
                    }
                }
            }
        }
    }

    post {
        always {
            // Define a etapa como sucesso, independentemente do resultado do script
            // O 'SUCCESS' pode ser substituído por outro status, se necessário.
            // Exemplos: ABORTED, UNSTABLE, FAILURE
            // Veja a documentação oficial para mais informações: 
            // https://www.jenkins.io/doc/book/pipeline/syntax/#post
            setBuildStatus(status: 'SUCCESS')
        }
    }
}

