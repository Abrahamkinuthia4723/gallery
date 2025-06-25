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
        ENABLE_EMAIL = "false"
    }

    stages {

        stage('Clone Repository') {
            steps {
                echo 'Cloning repository'
                git 'https://github.com/Abrahamkinuthia4723/gallery.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies'
                sh 'npm install'
            }
        }

        stage('Build Project') {
            steps {
                echo 'Building the project'
                sh 'npm run build'
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running tests'
                sh 'npm test'
            }
        }

        stage('Deploy to Render') {
            environment {
                NODE_ENV = 'production'
            }
            steps {
                echo 'Triggering deployment to Render'
                sh """
                    curl -X POST "${RENDER_DEPLOY_URL}?key=${RENDER_DEPLOY_KEY}"
                """
            }
        }

        stage('Send Slack Notification') {
            steps {
                echo 'Sending Slack notification'
                slackSend(
                    channel: "${SLACK_CHANNEL}",
                    message: "Build #${env.BUILD_ID} deployed successfully.\nProject URL: ${env.RENDER_URL}"
                )
            }
        }
    }

    post {
        failure {
            echo 'Build failed. Sending notifications.'

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

            slackSend(
                channel: "${SLACK_CHANNEL}",
                message: "Build #${env.BUILD_ID} FAILED.\nCheck logs: ${env.BUILD_URL}"
            )
        }
    }
}
