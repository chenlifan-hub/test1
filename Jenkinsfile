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
            def status = currentBuild.result ?: 'SUCCESS'
            def isSuccessful = (status == 'SUCCESS')
            def emoji = isSuccessful ? '✅' : '❌'
            def title = isSuccessful ? '构建成功' : '构建失败'

            def message = "${emoji} ${title}\n" +
                          "项目：${env.JOB_NAME}\n" +
                          "编号：#${env.BUILD_NUMBER}\n" +
                          "提交：${env.COMMIT_MESSAGE ?: '无'}\n" +
                          "详情：${env.BUILD_URL}"

            // 转义 JSON 特殊字符
            def escapedMessage = message
                .replace('\\', '\\\\')
                .replace('"', '\\"')
                .replace('\n', '\\n')

            // 从 Jenkins 凭据中读取飞书 webhook token
            withCredentials([string(credentialsId: 'c95c0f38-db1c-4175-b8ed-c17c32c5d65a', variable: 'FEISHU_WEBHOOK_TOKEN')]) {
                sh """
                    curl -X POST \\
                      -H 'Content-Type: application/json' \\
                      -d '{"msg_type":"text","content":{"text":"${escapedMessage}"}}' \\
                      "https://open.feishu.cn/open-apis/bot/v2/hook/\$FEISHU_WEBHOOK_TOKEN"
                """
            }
        }
    }
}
    

    
}
