stage('Package') {
    steps {
        sh '''
            set -eu

            rm -f ../website.tar.gz

            tar --exclude=.git \
                --exclude=.gitignore \
                -czf ../website.tar.gz .

            test -s ../website.tar.gz

            echo "Website package created successfully"
        '''
    }
}
