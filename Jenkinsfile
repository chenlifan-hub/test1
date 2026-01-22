pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.COMMIT_MESSAGE = sh(
                        script: 'git log -1 --pretty=%s',
                        returnStdout: true
                    ).trim()
                }
            }
        }

        stage('Build') {
            steps {
                sh 'echo "正在编译..."'
                sh 'ls'
            }
        }
    }

    post {
        always {
            script {
                def statusEmoji = currentBuild.result == 'SUCCESS' ? '✅' : '❌'
                def statusText = currentBuild.result ?: 'RUNNING'

                def message = """
Jenkins 构建通知
任务名称: ${env.JOB_NAME}
任务编号: #${env.BUILD_NUMBER}
状态: ${statusEmoji} ${statusText}
Commit: ${env.COMMIT_MESSAGE}
链接: ${env.BUILD_URL}
                """.stripIndent()

                // 发送纯文本消息（最简单）
                sh """
                    curl -X POST \\
                      -H 'Content-Type: application/json' \\
                      -d '{"msg_type":"text","content":{"text":"${message}"}}' \\
                      'https://open.feishu.cn/open-apis/bot/v2/hook/b56f684a-9c78-4aec-b525-4d1a2e8998cc'
                """
            }
        }
    }
}
