pipeline {
    agent any

    tools {
        nodejs "NodeJS 18"
    }

    stages {
        stage('Clone Code') {
            steps {
                git branch: 'master', url: 'https://github.com/Abrahamkinuthia4723/gallery.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Code') {
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
            withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK')]) {
                script {
                    def url = "https://gallery-1-cxp1.onrender.com"
                    def msg = "Build #${env.BUILD_ID} deployed successfully.\nProject URL: $url"
                    sh """curl -X POST -H "Content-type: application/json" --data '{"text": "${msg}"}' $SLACK_WEBHOOK"""
                }
            }
        }

        failure {
            withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK')]) {
                script {
                    def msg = "Build #${env.BUILD_ID} FAILED.\nCheck logs: ${env.BUILD_URL}"
                    sh """curl -X POST -H "Content-type: application/json" --data '{"text": "${msg}"}' $SLACK_WEBHOOK"""

                    def enableEmail = false
                    if (enableEmail) {
                        emailext(
                            subject: "Build Failed: #${env.BUILD_ID}",
                            body: "The build failed. Check logs:\n${env.BUILD_URL}",
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
