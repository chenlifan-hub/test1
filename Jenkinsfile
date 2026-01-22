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

            // 构造纯文本消息（避免 Markdown 特殊字符）
            def message = "Jenkins构建通知\n" +
                          "任务名称: ${env.JOB_NAME}\n" +
                          "任务编号: #${env.BUILD_NUMBER}\n" +
                          "状态: ${statusEmoji} ${statusText}\n" +
                          "Commit: ${env.COMMIT_MESSAGE}\n" +
                          "链接: ${env.BUILD_URL}"

            // 转义 JSON 中的特殊字符（主要是双引号、换行）
            def escapedMessage = message.replace('"', '\\"').replace('\n', '\\n')

            sh """
                curl -X POST \\
                  -H 'Content-Type: application/json' \\
                  -d '{"msg_type":"text","content":{"text":"${escapedMessage}"}}' \\
                  'https://open.feishu.cn/open-apis/bot/v2/hook/b56f684a-9c78-4aec-b525-4d1a2e8998cc'
            """
        }
    }
}
    

    
}
