pipeline {
    agent any

    tools {
        nodejs "NodeJS 18"
    }

    environment {
        RENDER_URL = "https://gallery-1-cxp1.onrender.com"
        RENDER_DEPLOY_URL = credentials('render-deploy-url')
        RENDER_DEPLOY_KEY = credentials('render-deploy-hook')
        SLACK_CHANNEL = "#Kinuthia_IP1"
        SLACK_WEBHOOK = credentials('slack-webhook')
        ENABLE_EMAIL = "false"
    }

    stages {
        stage('Clone code') {
            steps {
                echo 'Cloning repository...'
                git branch: 'master', url: 'https://github.com/Abrahamkinuthia4723/gallery.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh 'npm install'
            }
        }

        stage('Build code') {
            steps {
                echo 'Building code...'
                sh 'npm run build'
            }
        }

        stage('Test code') {
            steps {
                echo 'Running tests...'
                sh 'npm test'
            }
        }

        stage('Deploy to Render') {
            environment {
                NODE_ENV = 'production'
            }
            steps {
                echo 'Triggering deployment to Render...'
                sh '''
                    curl -X POST "$RENDER_DEPLOY_URL?key=$RENDER_DEPLOY_KEY"
                '''
            }
        }
    }

    post {
        success {
            echo 'Build succeeded. Sending Slack notification...'
            sh '''
                curl -X POST -H "Content-type: application/json" \
                --data "{\"text\":\"Build #$BUILD_ID deployed successfully.\\nProject URL: $RENDER_URL\"}" \
                $SLACK_WEBHOOK
            '''
        }

        failure {
            echo 'Build failed. Sending notifications...'
            script {
                if (env.ENABLE_EMAIL == 'true') {
                    emailext(
                        subject: "Jenkins Build Failed: #${env.BUILD_ID}",
                        body: """The build has failed.

Project URL: ${env.RENDER_URL}
Build URL: ${env.BUILD_URL}""",
                        to: "kinuthia.abraham@student.moringaschool.com",
                        mimeType: 'text/plain',
                        attachLog: true
                    )
                }
            }
            sh '''
                curl -X POST -H "Content-type: application/json" \
                --data "{\"text\":\"Build #$BUILD_ID FAILED.\\nCheck logs: $BUILD_URL\"}" \
                $SLACK_WEBHOOK
            '''
        }
    }
}
