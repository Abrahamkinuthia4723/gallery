pipeline {
    agent any

    tools {
        nodejs "NodeJS 18"
    }

    environment {
        RENDER_URL = "https://gallery-1-cxp1.onrender.com"
        RENDER_DEPLOY_URL = "https://api.render.com/deploy/srv-d1cpa13ipnbc73c215r0"
        SLACK_CHANNEL = "#Kinuthia_IP1"
        RENDER_DEPLOY_KEY = credentials('render-deploy-hook')
        SLACK_WEBHOOK = credentials('slack-webhook')
        ENABLE_EMAIL = "false"
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
            }
            steps {
                sh """
                    curl -X POST "${RENDER_DEPLOY_URL}?key=${RENDER_DEPLOY_KEY}"
                """
            }
        }

        stage('Send Slack Notification') {
            steps {
                sh """
                    curl -X POST -H 'Content-type: application/json' \
                    --data '{"text":"Build #${env.BUILD_ID} deployed successfully.\\nProject URL: ${env.RENDER_URL}"}' \
                    ${SLACK_WEBHOOK}
                """
            }
        }
    }

    post {
        success {
            sh """
                curl -X POST -H 'Content-type: application/json' \
                --data '{"text":"Build #${env.BUILD_ID} SUCCESS.\\nProject URL: ${env.RENDER_URL}"}' \
                ${SLACK_WEBHOOK}
            """
        }

        failure {
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

            sh """
                curl -X POST -H 'Content-type: application/json' \
                --data '{"text":"Build #${env.BUILD_ID} FAILED.\\nCheck logs: ${env.BUILD_URL}"}' \
                ${SLACK_WEBHOOK}
            """
        }
    }
}
