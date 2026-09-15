pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    tools {
        maven 'Maven-3.9.12'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Code checked out from GitHub'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Create ZIP') {
            steps {
                sh 'zip -j devops-project.zip index.html'
            }
        }

        stage('Deploy App 1') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'app1-ubuntu-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                        unzip -p devops-project.zip index.html \
                        | sed \
                        -e 's/SERVER_NAME/dev-webserver-1/g' \
                        -e 's/SERVER_IP/10.0.0.58/g' \
                        -e 's/SERVER_SUBNET/Private-subnet-dev-1A/g' \
                        -e 's/SERVER_AZ/eu-west-1a/g' \
                        -e 's/SERVER_OS/Ubuntu/g' \
                        > webserver1.html

                        scp -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            -i "$SSH_KEY" \
                            webserver1.html \
                            "$SSH_USER@10.0.0.58:/tmp/webserver1.html"

                        ssh -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            -i "$SSH_KEY" \
                            "$SSH_USER@10.0.0.58" \
                            'sudo mv /tmp/webserver1.html /var/www/html/webserver1.html && sudo chmod 644 /var/www/html/webserver1.html'

                        rm -f webserver1.html
                    '''
                }
            }
        }

        stage('Deploy App 2') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'app2-amazonlinux-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                        unzip -p devops-project.zip index.html \
                        | sed \
                        -e 's/SERVER_NAME/dev-webserver-2/g' \
                        -e 's/SERVER_IP/10.0.0.126/g' \
                        -e 's/SERVER_SUBNET/Private-subnet-dev-1B/g' \
                        -e 's/SERVER_AZ/eu-west-1b/g' \
                        -e 's/SERVER_OS/Amazon Linux 2023/g' \
                        > webserver2.html

                        scp -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            -i "$SSH_KEY" \
                            webserver2.html \
                            "$SSH_USER@10.0.0.126:/tmp/webserver2.html"

                        ssh -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            -i "$SSH_KEY" \
                            "$SSH_USER@10.0.0.126" \
                            'sudo mv /tmp/webserver2.html /usr/share/nginx/html/webserver2.html && sudo chmod 644 /usr/share/nginx/html/webserver2.html'

                        rm -f webserver2.html
                    '''
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'devops-project.zip', fingerprint: true
        }
    }
}
