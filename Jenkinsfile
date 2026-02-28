pipeline{
    agent any
    // agent{
    //     docker{
    //             image 'node:18-alpine'
    //             reuseNode true
    //         }
    //     }

    stages{
        // stage('Build'){
            
        //     steps{
        //         sh '''
        //             ls -la
        //             node --version
        //             npm --version
        //             npm ci
        //             npm run build
        //         '''
        //     }
        // }
        stage('Test'){
            agent{
                docker{
                        image 'node:18-alpine'
                        reuseNode true
                    }
                }
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
        stage('E2E'){
            agent{
                docker{
                        image 'mcr.microsoft.com/playwright:v1.58.2-noble'
                        reuseNode true
                    }
                }
            steps{
                sh '''
                    npm install -g serve
                    serve -s build 
                    npx playright test
                '''
            }
        }
        
    }
    post{
            always{
                junit 'test-results/junit.xml'
            }
        }

}