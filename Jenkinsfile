pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        WEB_SERVER = 'WEB_SERVER_PRIVATE_IP'
        DEPLOY_USER = 'deploy'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate') {
            steps {
                sh '''
                    set -eu
                    test -f index.html
                    test -f style.css
                    echo "Website files validated successfully"
                '''
            }
        }

        stage('Package') {
            steps {
                sh '''
                    set -eu
                    rm -f website.tar.gz
                    tar -czf website.tar.gz index.html style.css
                    test -s website.tar.gz
                    ls -lh website.tar.gz
                    echo "Package created successfully"
                '''
            }
        }

        stage('Deploy') {
            steps {
                sshagent(credentials: ['webserver-deploy-key']) {
                    sh '''
                        set -eu

                        ssh -o BatchMode=yes \
                            "$DEPLOY_USER@$WEB_SERVER" \
                            'test -w /var/www/html'

                        scp -o BatchMode=yes website.tar.gz \
                            "$DEPLOY_USER@$WEB_SERVER:/tmp/website-${BUILD_NUMBER}.tar.gz"

                        ssh -o BatchMode=yes \
                            "$DEPLOY_USER@$WEB_SERVER" \
                            "tar -xzf /tmp/website-${BUILD_NUMBER}.tar.gz -C /var/www/html && rm -f /tmp/website-${BUILD_NUMBER}.tar.gz"

                        echo "Deployment completed successfully"
                    '''
                }
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    set -eu
                    curl --fail --silent --show-error \
                        "http://$WEB_SERVER/" \
                        -o /tmp/deployed-index.html

                    grep -q "Automated CI/CD Pipeline" /tmp/deployed-index.html
                    echo "Website verification successful"
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: CI/CD pipeline completed successfully.'
        }
        failure {
            echo 'FAILED: Check the first failing stage in Console Output.'
        }
    }
}
