pipeline {
    agent any

    tools {
        nodejs "NodeJS 18"
    }

    environment {
        RENDER_URL = "https://gallery-1-cxp1.onrender.com"
        RENDER_DEPLOY_URL = credentials('render-deploy-url')
        RENDER_DEPLOY_KEY = credentials('render-deploy-hook')
        SLACK_WEBHOOK = credentials('slack-webhook')
        ENABLE_EMAIL = "false"
        NODE_ENV = "production"
    }

    stages {
        stage('Clone') {
            steps {
                git branch: 'master', url: 'https://github.com/Abrahamkinuthia4723/gallery.git'
            }
        }

        stage('Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Deploy') {
            steps {
                sh "curl -X POST '$RENDER_DEPLOY_URL?key=$RENDER_DEPLOY_KEY'"
            }
        }
    }

    post {
        success {
            sh """
                curl -X POST -H "Content-type: application/json" \
                --data '{"text":"Build #$BUILD_ID succeeded and deployed. \\nProject: $RENDER_URL"}' \
                $SLACK_WEBHOOK
            """
        }

        failure {
            script {
                if (env.ENABLE_EMAIL == 'true') {
                    emailext(
                        subject: "Build Failed: #${env.BUILD_ID}",
                        body: "The build has failed.\nProject: ${env.RENDER_URL}\nLogs: ${env.BUILD_URL}",
                        to: "kinuthia.abraham@student.moringaschool.com",
                        mimeType: 'text/plain',
                        attachLog: true
                    )
                }
            }
            sh """
                curl -X POST -H "Content-type: application/json" \
                --data '{"text":"Build #$BUILD_ID FAILED. \\nLogs: $BUILD_URL"}' \
                $SLACK_WEBHOOK
            """
        }
    }
}
