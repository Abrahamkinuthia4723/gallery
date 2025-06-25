pipeline {
    agent any

    tools {
        nodejs "NodeJS 18"
    }

    environment {
        NODE_ENV = "production"
        RENDER_URL = "https://gallery-1-cxp1.onrender.com"
        SLACK_CHANNEL = "#Kinuthia_IP1"
        SLACK_WEBHOOK = credentials('slack-webhook')
        RENDER_DEPLOY_KEY = credentials('render-deploy-hook')
        ENABLE_EMAIL = "false"
    }

    stages {

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
            steps {
                echo 'Triggering deployment to Render'
                sh """
                    curl -X POST "https://api.render.com/deploy/srv-d1cpa13ipnbc73c215r0?key=${RENDER_DEPLOY_KEY}"
                """
            }
        }

        stage('Send Slack Notification') {
            steps {
                echo 'Sending Slack notification'
                sh """
                    curl -X POST -H 'Content-type: application/json' \
                    --data '{"text": "Build #${env.BUILD_ID} deployed successfully. Project URL: ${env.RENDER_URL}"}' \
                    "${env.SLACK_WEBHOOK}"
                """
            }
        }
    }

    post {
        failure {
            echo 'Build failed. Sending notifications.'

            script {
                if (env.ENABLE_EMAIL == 'true') {
                    mail to: 'kinuthia.abraham@student.moringaschool.com',
                         subject: "Jenkins Build Failed: Build #${env.BUILD_ID}",
                         body: """
The build has failed.

Check the Jenkins logs for more details.

Project URL: ${env.RENDER_URL}
Build URL: ${env.BUILD_URL}
                         """
                }
            }

            node {
                sh """
                    curl -X POST -H 'Content-type: application/json' \
                    --data '{"text": "Build #${env.BUILD_ID} FAILED. Check logs: ${env.BUILD_URL}"}' \
                    "${env.SLACK_WEBHOOK}"
                """
            }
        }
    }
}
