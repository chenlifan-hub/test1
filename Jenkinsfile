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

                // 1. 序列化 JSON
                def jsonPayload = groovy.json.JsonOutput.toJson(payload)

                // 2. 获取当前时间戳（秒）
                def timestamp = System.currentTimeMillis() / 1000

                // 3. 获取签名密钥（从 Jenkins Credentials）
                def signingSecret = credentials(env.FEISHU_SIGNING_SECRET_ID)

                // 4. 构造签名字符串：timestamp + "\n" + body
                def signStr = "${timestamp}\n${jsonPayload}"

                // 5. 计算 HMAC-SHA256 签名（Base64 编码）
                def hmac = javax.crypto.Mac.getInstance("HmacSHA256")
                def secretKey = new javax.crypto.spec.SecretKeySpec(signingSecret.bytes, "HmacSHA256")
                hmac.init(secretKey)
                def signature = hmac.doFinal(signStr.bytes).encodeBase64().toString()

                // 6. 发送带签名的请求
                sh """
                    curl -X POST \\
                      -H 'Content-Type: application/json' \\
                      -H 'X-Lark-Timestamp: ${timestamp}' \\
                      -H 'X-Lark-Signature: ${signature}' \\
                      -d '${jsonPayload}' \\
                      '${env.FEISHU_WEBHOOK}'
                """
            }
        }
    }
}
