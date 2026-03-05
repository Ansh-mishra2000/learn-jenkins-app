pipeline {
    agent any

	environment {
		NETLIFY_SITE_ID= 'beb46202-fbdf-464e-8638-07f564e09a36'	
		NETLIFY_AUTH_TOKEN= credentials('netlify-token')
        
	}

    // agent {
    //     docker {
    //         image 'node:18-alpine'
    //         reuseNode true
    //     }
    // }

    stages {

    stage('Build') {
        agent{
            docker {
                image 'node:18-alpine'
                reuseNode true
            }
        }
        steps {
            sh '''
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
            '''
        }
    }

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
									reportName: 'Playwright Local Report',
									reportTitles: '',
									useWrapperFileDirectly: true
								])
							}
   						}
                }

            }
        }
		stage('Deploy') {
			agent{
				docker {
					image 'node:18'
					reuseNode true
				}
			}
            steps {
                sh '''
                    npm install netlify-cli
					node_modules/.bin/netlify --version
					echo "Deploying to production. Site Id: $NETLIFY_SITE_ID"
					node_modules/.bin/netlify status
					node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
        stage('Deploying staging') {
                agent{
                    docker {
                        image 'mcr.microsoft.com/playwright:v1.58.2-noble'
                        reuseNode true
                    }
                }

                steps {
                    sh '''
                    npm install netlify-cli node-jq
					node_modules/.bin/netlify --version
					echo "Deploying to production. Site Id: $NETLIFY_SITE_ID"
					node_modules/.bin/netlify status
					node_modules/.bin/netlify deploy --dir=build --json > deploy-out.txt
					node_modules/.bin/node-jq -r '.deploy_url' deploy-out.txt
                '''
                script{
                    env.STAGING_URL=sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-out.txt",returnStdout:true);
                }
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
                                reportName: 'Playwright E2E',
                                reportTitles: '',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
            }
        stage('Approval') {
            steps {
                input message: 'Approve Production E2E deployment?', ok: 'Deploy'
            }
        }

        stage('Staging E2E') {
                agent {
                    docker {
                        image 'mcr.microsoft.com/playwright:v1.58.2-noble'
                        reuseNode true
                    }
                }

                environment {
                    CI_ENVIRONMENT_URL="$env.STAGING_URL"  //or "${env.STAGING_URL}"
                }
                

                steps {
                //     timeout(time: 10, unit: 'MINUTES') {
                //         input message: 'Approve running Production E2E tests?', ok: 'Run Tests'
                // }
                    sh '''
                         
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
                                reportName: 'Staging E2E',
                                reportTitles: '',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
        }


        stage('Deploy pro E2E') {
                agent {
                    docker {
                        image 'mcr.microsoft.com/playwright:v1.58.2-noble'
                        reuseNode true
                    }
                }
                environment {
                    CI_ENVIRONMENT_URL='https://snazzy-alfajores-8d226b.netlify.app'
        }
                

                // steps {
                //     timeout(time: 10, unit: 'MINUTES') {
                //         input message: 'Approve running Production E2E tests?', ok: 'Run Tests'
                // }
                    sh '''
                         
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
                                reportName: 'Production E2E',
                                reportTitles: '',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
            }

        }