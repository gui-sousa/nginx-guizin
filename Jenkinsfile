pipeline {
    agent any
    environment {
        POST_CREDS = credentials('N1SOrlYIEbZOBWUduz7Wj7rRW')
    }

    stages {
        stage('Atualizando Código') {
            steps {
                git branch: 'master', url: 'https://github.com/gui-sousa/nginx-guizin.git'
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
                sh 'chmod u+x ./kubectl'
                sh './kubectl apply -f https://raw.githubusercontent.com/gui-sousa/nginx-guizin/master/service.yaml'
                sh './kubectl apply -f https://raw.githubusercontent.com/gui-sousa/nginx-guizin/master/deployment.yaml'
            }
        }

      }

      stage('Test Nginx Page') {
        steps {
            httpRequest consoleLogResponseBody: true, responseHandle: 'NONE', url: 'http://10.1.81.21:32001/', validResponseCodes: '200', validResponseContent: 'Guizin!'
        }

      }  
    }

    post {
        success {
            hangoutsNotify message: "✔ Deu Certo!", token: "${POST_CREDS}", threadByJob: false
        }

        failure {
           hangoutsNotify message: "❌ Deu Errado!", token: "${POST_CREDS}", threadByJob: false 
        }
    }
}

