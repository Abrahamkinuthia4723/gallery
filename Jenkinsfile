pipeline {
    agent any

    tools {
        nodejs "NodeJS 18"
    }

    stages {
        stage('Clone code') {
            steps {
                git branch: 'master', url: 'https://github.com/Abrahamkinuthia4723/gallery.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build code') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Test code') {
            steps {
                sh 'npm test'
            }
        }

        stage('Deploy to Render') {
            environment {
                NODE_ENV = 'production'
                RENDER_DEPLOY_URL = credentials('render-deploy-url')
                RENDER_DEPLOY_KEY = credentials('render-deploy-hook')
            }
            steps {
                sh 'curl -X POST "$RENDER_DEPLOY_URL?key=$RENDER_DEPLOY_KEY"'
            }
        }
    }

    post {
        success {
            steps {
                withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK')]) {
                    script {
                        def msg = "Build #${env.BUILD_ID} deployed successfully. Project URL: https://gallery-1-cxp1.onrender.com"
                        sh """
                            curl -X POST -H "Content-type: application/json" \
                            --data '{"text":"${msg}"}' $SLACK_WEBHOOK
                        """
                    }
                }
            }
        }

        failure {
            steps {
                withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK')]) {
                    script {
                        def msg = "Build #${env.BUILD_ID} FAILED. Check logs: ${env.BUILD_URL}"
                        sh """
                            curl -X POST -H "Content-type: application/json" \
                            --data '{"text":"${msg}"}' $SLACK_WEBHOOK
                        """

                        def enableEmail = false
                        if (enableEmail) {
                            emailext(
                                subject: "Jenkins Build Failed: #${env.BUILD_ID}",
                                body: "The build has failed.\n\nBuild URL: ${env.BUILD_URL}",
                                to: "kinuthia.abraham@student.moringaschool.com",
                                mimeType: 'text/plain',
                                attachLog: true
                            )
                        }
                    }
                }
            }
        }
    }
}
