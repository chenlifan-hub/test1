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

            withCredentials([string(credentialsId: 'c95c0f38-db1c-4175-b8ed-c17c32c5d65a', variable: 'FEISHU_SECRET')]) {
                // 将 JSON 写入临时文件
                writeFile file: "${env.WORKSPACE}/feishu_payload.json", text: jsonPayload
                
                // 使用纯 Shell 脚本计算签名（避免 Groovy/Shell 变量混合）
                sh """
                    #!/bin/bash
                    TIMESTAMP=${timestamp}
                    SECRET="${FEISHU_SECRET}"
                    
                    # 读取 JSON 内容
                    JSON_CONTENT=\$(cat '${env.WORKSPACE}/feishu_payload.json')
                    
                    # 构造签名字符串（注意：不需要额外换行，openssl 会自动处理）
                    SIGN_STRING="\${TIMESTAMP}\\\\n\${JSON_CONTENT}"
                    
                    # 计算签名
                    # 使用 printf 避免添加额外的换行符
                    SIGNATURE=\$(printf "%s" "\$SIGN_STRING" | openssl dgst -sha256 -hmac "\$SECRET" -binary | base64)
                    
                    echo "Timestamp: \${TIMESTAMP}"
                    echo "Signature: \${SIGNATURE}"
                    
                    # 发送请求
                    curl -X POST \\
                         -H 'Content-Type: application/json' \\
                         -H "X-Lark-Timestamp: \${TIMESTAMP}" \\
                         -H "X-Lark-Signature: \${SIGNATURE}" \\
                         -d "@${env.WORKSPACE}/feishu_payload.json" \\
                         '${FEISHU_WEBHOOK}' \\
                         --verbose
                    
                    # 清理临时文件
                    rm -f '${env.WORKSPACE}/feishu_payload.json'
                """
            }
        }
    }
}

      
}
