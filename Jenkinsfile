pipeline {
    agent any

    environment {
        // Eğer Docker Hub’a push edecekseniz, repo adını değiştirin
        DOCKER_REPO = "egitimicerigim/blog"
    }

    stages {
        stage('Checkout') {
            steps {
                // GitHub deposundan kodu çek
                git branch: 'main',
                    url: 'https://github.com/egitimicerigim/jenkins.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Version olarak BUILD_ID kullanıyoruz (her build'de farklı tag)
                    def dockerImage = docker.build("${DOCKER_REPO}:${env.BUILD_ID}")
                    // Ortam değişkeni olarak saklayalım ki sonraki adımlarda kullanabilelim
                    env.IMAGE_NAME = "${DOCKER_REPO}:${env.BUILD_ID}"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Docker registry kimlik bilgilerini kullanarak
                    docker.withRegistry('', 'docker-credentials-id') {
                        sh "docker push ${env.IMAGE_NAME}"
                    }
                }
            }
        }

        stage('Deploy to Remote Server') {
            steps {
                // SSH kimlik bilgilerini kullanarak remote sunucuya bağlan
                sshagent(['remote-ssh-credentials']) {
                    // Bağlandığımız sunucuda imajı pull edip container'ı çalıştırıyoruz.
                    // Burada remote sunucuda var olan bir container varsa önce stop/remove
                    // etmeniz gerekebilir. Örnek olması açısından basit bir yaklaşım gösterildi.
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@10.0.2.15 'docker pull ${env.IMAGE_NAME}'
                    ssh -o StrictHostKeyChecking=no ubuntu@10.0.2.15 'docker stop my-sample-app || true && docker rm my-sample-app || true'
                    ssh -o StrictHostKeyChecking=no ubuntu@10.0.2.15 'docker run -d --name my-sample-app -p 80:80 ${env.IMAGE_NAME}'
                    """
                }
            }
        }
    }
}
