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
            // ✅ 关键修复：强制转为整数秒（去掉小数）
            def timestamp = (System.currentTimeMillis() / 1000).toInteger()

            withCredentials([string(credentialsId: 'c95c0f38-db1c-4175-b8ed-c17c32c5d65a', variable: 'FEISHU_SIGN')]) {
                def tempJsonFile = "${env.WORKSPACE}/feishu_payload.json"
                sh "echo '${jsonPayload}' > '${tempJsonFile}'"

                sh """
                    JSON_CONTENT=\$(cat '${tempJsonFile}')
                    # ✅ 注意：这里 timestamp 已是整数，直接拼接
                    SIGN_STR="${timestamp}\\n\${JSON_CONTENT}"
                    
                    # 使用 printf 确保无额外换行
                    SIGNATURE=\$(printf '%s' "\$SIGN_STR" | openssl dgst -sha256 -hmac "\$FEISHU_SIGN" -binary | base64)
                    
                    curl -X POST \\
                      -H 'Content-Type: application/json' \\
                      -H 'X-Lark-Timestamp: ${timestamp}' \\
                      -H "X-Lark-Signature: \$SIGNATURE" \\
                      -d "@${tempJsonFile}" \\
                      'https://open.feishu.cn/open-apis/bot/v2/hook/b56f684a-9c78-4aec-b525-4d1a2e8998cc'
                    
                    rm -f '${tempJsonFile}'
                """
            }
        }
    }
}
      
}
