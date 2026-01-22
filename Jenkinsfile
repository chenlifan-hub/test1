pipeline {
    agent any

    environment {
        // 飞书 Webhook 地址（你提供的）
        FEISHU_WEBHOOK = 'https://open.feishu.cn/open-apis/bot/v2/hook/b56f684a-9c78-4aec-b525-4d1a2e8998cc'
        // Jenkins 凭据 ID（你创建的 Secret Text）
        FEISHU_SIGNING_SECRET_ID = 'c95c0f38-db1c-4175-b8ed-c17c32c5d65a'
    }

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
            ## Jenkins 构建通知
            - 任务名称: ${env.JOB_NAME}
            - 任务编号: #${env.BUILD_NUMBER}
            - ${statusEmoji} 构建状态: ${statusText}
            - Commit message: ${env.COMMIT_MESSAGE}
            - 链接: [查看详情](${env.BUILD_URL})
            """

            def payload = [
                msg_type: 'interactive',
                card: [
                    config: [wide_screen_mode: true],
                    header: [
                        title: [tag: 'plain_text', content: "构建 ${statusText}"],
                        template: currentBuild.result == 'SUCCESS' ? 'green' : 'red'
                    ],
                    elements: [
                        [
                            tag: 'div',
                            text: [tag: 'lark_md', content: message]
                        ]
                    ]
                ]
            ]

            def jsonPayload = groovy.json.JsonOutput.toJson(payload)
            def timestamp = System.currentTimeMillis() / 1000

            // 从凭据获取签名密钥（自动注入到环境变量更安全）
            withCredentials([string(credentialsId: 'c95c0f38-db1c-4175-b8ed-c17c32c5d65a', variable: 'FEISHU_SIGN')]) {
                sh """
                    # 准备签名字符串：timestamp + "\\n" + body
                    SIGN_STR="${timestamp}\\n${jsonPayload}"

                    # 使用 openssl 计算 HMAC-SHA256 并 Base64 编码
                    SIGNATURE=\$(echo -n "\$SIGN_STR" | openssl dgst -sha256 -hmac "\$FEISHU_SIGN" -binary | base64)

                    # 发送请求
                    curl -X POST \\
                      -H 'Content-Type: application/json' \\
                      -H "X-Lark-Timestamp: ${timestamp}" \\
                      -H "X-Lark-Signature: \$SIGNATURE" \\
                      -d '${jsonPayload}' \\
                      'https://open.feishu.cn/open-apis/bot/v2/hook/b56f684a-9c78-4aec-b525-4d1a2e8998cc'
                """
            }
        }
    }
}
      
}
