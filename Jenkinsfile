pipeline {
    agent any

    environment {
        CHAT_TOKEN = credentials('google-chat-guisousa')
        APP_ADRESS = "http://10.1.81.21:32001/"
    }

    stages {
        stage('Atualizando Código') {
            steps {
                git branch: 'master', url: 'https://github.com/gui-sousa/nginx-guizin.git'
            }

            post {
                always {
                   hangoutsNotify message: "⚙️ Iniciando build\n⏰ Horario de início: $BUILD_TIMESTAMP", token: "$CHAT_TOKEN", threadByJob: false
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    dockerapp = docker.build("guisousa/nginx-guizin:${env.BUILD_ID}", "-f Dockerfile .")
                }
            }
        }

        stage('Docker Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-gsousa') {
                        dockerapp.push('latest')
                        dockerapp.push(env.BUILD_ID)
                    }
                }
            }
        }

        stage('Deploy Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'k0s-vanuatu']) {
                    sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.20.5/bin/linux/amd64/kubectl"'  
                    sh './kubectl apply -f service.yaml'
                    sh './kubectl apply -f deployment.yaml'
                }
            }
        }

        stage('Test Nginx Page') {
            steps {
                httpRequest consoleLogResponseBody: true, responseHandle: 'NONE', url: "$APP_ADRESS", validResponseCodes: '200', validResponseContent: 'Guizin!'
            }
        }
    }

    post {
        success {
            hangoutsNotify message: "✅ Deu Certo!\n⏰ Tempo de Duração: ${currentBuild.duration / 1000} segundos", token: "$CHAT_TOKEN", threadByJob: false
        }

        failure {
           hangoutsNotify message: "❌ Deu Errado!\n⏰ Tempo de Duração: ${currentBuild.duration / 1000} segundos", token: "$CHAT_TOKEN", threadByJob: false 
        }
    }
}
