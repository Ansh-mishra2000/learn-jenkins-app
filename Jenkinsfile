pipeline {
    agent any

    // agent {
    //     docker {
    //         image 'node:18-alpine'
    //         reuseNode true
    //     }
    // }

    stages {

        // stage('Build') {
        //     steps {
        //         sh '''
        //             ls -la
        //             node --version
        //             npm --version
        //             npm ci
        //             npm run build
        //         '''
        //     }
        // }

        stage('Test') {

            parallel {

                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            if [ -f build/index.html ]; then
                                echo "file exists"
                            else
                                echo "file does not exist"
                            fi
                            npm test
                        '''
                    }

					post {
        				always {
            					junit 'jest-results/junit.xml'
							}
   						}
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.58.2-noble'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 3
                            npx playwright test --reporter=html
                        '''
                    }
					post {
        				always {
								publishHTML([
									allowMissing: false,
									alwaysLinkToLastBuild: false,
									icon: '',
									keepAll: false,
									reportDir: 'playwright-report',
									reportFiles: 'index.html',
									reportName: 'HTML Report',
									reportTitles: '',
									useWrapperFileDirectly: true
								])
							}
   						}
                }

            }
        }
    }


    
}