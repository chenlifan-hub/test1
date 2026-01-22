pipeline {
    agent any

    environment {
        // 飞书 Webhook 地址
        FEISHU_WEBHOOK = 'https://open.feishu.cn/open-apis/bot/v2/hook/b56f684a-9c78-4aec-b525-4d1a2e8998cc'
        // Jenkins 凭据 ID（存储签名密钥）
        FEISHU_SECRET_ID = 'c95c0f38-db1c-4175-b8ed-c17c32c5d65a'
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
                
                // 将 JSON 写入临时文件
                writeFile file: "${env.WORKSPACE}/feishu_payload.json", text: jsonPayload

                withCredentials([string(credentialsId: env.FEISHU_SECRET_ID, variable: 'SECRET')]) {
                    // 方法1：使用纯 Shell 计算签名（推荐）
                    sh """
                        #!/bin/bash
                        TIMESTAMP=${timestamp}
                        
                        # 根据飞书文档：签名 = base64(hmac_sha256(timestamp + "\\n" + secret))
                        SIGN_STRING="\${TIMESTAMP}\\n"
                        SIGNATURE=\$(echo -n "\$SIGN_STRING\${SECRET}" | openssl dgst -sha256 -binary | base64)
                        
                        echo "=== 调试信息 ==="
                        echo "Timestamp: \${TIMESTAMP}"
                        echo "Secret: \${SECRET}"
                        echo "Sign String: \${SIGN_STRING}\${SECRET}"
                        echo "Signature: \${SIGNATURE}"
                        echo "================"
                        
                        # 发送请求到飞书
                        RESPONSE=\$(curl -s -w "\\nHTTP_STATUS:%{http_code}" \\
                            -X POST \\
                            -H 'Content-Type: application/json' \\
                            -H "X-Lark-Timestamp: \${TIMESTAMP}" \\
                            -H "X-Lark-Signature: \${SIGNATURE}" \\
                            -d @'${env.WORKSPACE}/feishu_payload.json' \\
                            '${FEISHU_WEBHOOK}')
                        
                        echo "飞书响应: \$RESPONSE"
                        
                        # 清理临时文件
                        rm -f '${env.WORKSPACE}/feishu_payload.json'
                    """
                    
                    // 方法2：使用 Groovy 计算签名（备用方案）
                    /*
                    script {
                        import javax.crypto.Mac
                        import javax.crypto.spec.SecretKeySpec
                        import java.security.InvalidKeyException
                        
                        // 构造签名字符串：timestamp + "\n" + secret
                        def signString = "${timestamp}\\n${SECRET}"
                        
                        // 计算 HMAC-SHA256
                        def secretKeySpec = new SecretKeySpec(SECRET.getBytes("UTF-8"), "HmacSHA256")
                        def mac = Mac.getInstance("HmacSHA256")
                        mac.init(secretKeySpec)
                        def signature = mac.doFinal(signString.getBytes("UTF-8"))
                        
                        // Base64 编码
                        def encodedSignature = signature.encodeBase64().toString().trim()
                        
                        echo "Groovy 计算签名: ${encodedSignature}"
                        
                        // 发送请求
                        def result = sh(script: """
                            curl -X POST \\
                                 -H 'Content-Type: application/json' \\
                                 -H "X-Lark-Timestamp: ${timestamp}" \\
                                 -H "X-Lark-Signature: ${encodedSignature}" \\
                                 -d @'${env.WORKSPACE}/feishu_payload.json' \\
                                 '${FEISHU_WEBHOOK}'
                        """, returnStdout: true)
                        
                        echo "飞书响应: ${result}"
                    }
                    */
                }
            }
        }
    }
}
