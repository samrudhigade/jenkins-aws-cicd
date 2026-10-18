
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        WEB_SERVER = '172.31.37.164'
        DEPLOY_USER = 'deploy'
        WEB_ROOT = '/var/www/html'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub'
                checkout scm
            }
        }

        stage('Validate') {
            steps {
                sh '''
                    set -eu
                    echo "Workspace: $PWD"
                    ls -la

                    if [ ! -f index.html ]; then
                        echo "ERROR: index.html is missing"
                        exit 1
                    fi
                '''
            }
        }

        stage('Package') {
            steps {
                sh '''
                    set -eu

                    PACKAGE_FILE=$(mktemp /tmp/website-XXXXXX.tar.gz)

                    tar --exclude='./.git' \
                        --exclude='./.gitignore' \
                        -czf "$PACKAGE_FILE" .

                    mv "$PACKAGE_FILE" website.tar.gz
                    test -s website.tar.gz

                    echo "Website package created successfully"
                    ls -lh website.tar.gz
                '''
            }
        }

        stage('Deploy') {
            steps {
                sshagent(credentials: ['webserver-deploy-key']) {
                    sh '''
                        set -eu

                        mkdir -p ~/.ssh
                        chmod 700 ~/.ssh

                        ssh-keyscan -H "$WEB_SERVER" >> ~/.ssh/known_hosts
                        chmod 600 ~/.ssh/known_hosts

                        ssh -o BatchMode=yes \
                            "$DEPLOY_USER@$WEB_SERVER" \
                            "test -w '$WEB_ROOT'"

                        scp website.tar.gz \
                            "$DEPLOY_USER@$WEB_SERVER:/tmp/website-${BUILD_NUMBER}.tar.gz"

                        ssh "$DEPLOY_USER@$WEB_SERVER" \
                            "tar -xzf /tmp/website-${BUILD_NUMBER}.tar.gz -C '$WEB_ROOT' &&
                             rm -f /tmp/website-${BUILD_NUMBER}.tar.gz"

                        echo "Deployment completed"
                    '''
                }
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    set -eu

                    curl --fail --silent --show-error \
                        --max-time 15 \
                        "http://$WEB_SERVER/" \
                        -o /tmp/deployed-page.html

                    echo "Website HTTP check passed"
                    head -n 5 /tmp/deployed-page.html
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Website deployed and HTTP verification passed.'
        }
        failure {
            echo 'FAILED: Check the stage logs in Console Output.'
        }
        always {
            echo 'Pipeline execution finished.'
        }
    }
}

