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
    }

    stages {
        stage('Install Dependencies') {
            steps {
                echo '📦 Installing dependencies...'
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                echo '🔨 Building the project...'
                sh 'npm run build || echo "⚠️ No build script found, skipping..."'
            }
        }

        stage('Test') {
            steps {
                echo '🧪 Running tests...'
                sh 'npm test'
            }
        }

        stage('Deploy to Render') {
            steps {
                echo '🚀 Triggering Render deploy...'
                sh 'curl -X POST https://api.render.com/deploy/srv-d1cpa13ipnbc73c215r0?key=lQznZCqpkmE'
            }
        }

        stage('Slack Notification') {
            steps {
                echo '📢 Sending Slack notification...'
                sh """
                    curl -X POST -H 'Content-type: application/json' \
                    --data '{"text": "✅ *Build #${env.BUILD_ID}* deployed successfully!\\n🔗 ${env.RENDER_URL}"}' \
                    "${env.SLACK_WEBHOOK}"
                """
            }
        }
    }

    post {
        failure {
            echo '❌ Build failed. Sending notifications...'

            mail to: 'kinuthia.abraham@student.moringaschool.com',
                 subject: "❌ Jenkins Build Failed: Build #${env.BUILD_ID}",
                 body: """Build failed.
Check Jenkins logs to debug the issue.

Project URL: ${env.RENDER_URL}
Build URL: ${env.BUILD_URL}"""

            sh """
                curl -X POST -H 'Content-type: application/json' \
                --data '{"text": "❌ *Build #${env.BUILD_ID} FAILED!*\\nCheck logs: ${env.BUILD_URL}"}' \
                "${env.SLACK_WEBHOOK}"
            """
        }
    }
}
