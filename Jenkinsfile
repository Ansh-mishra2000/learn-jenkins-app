pipeline{
    agent{
        docker{
                image 'node:18-alpine'
                reuseNode true
            }
        }

    stages{
        stage('Build'){
            
            steps{
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        }
        stage('Test'){
            steps{
                sh '''
                    if [ -f build/index.html ]; then  # space after [ is important
                        echo "file exists"
                    else
                        echo "file does not exist"
                    fi
                    npm test
                '''
            }
        }
    }
}