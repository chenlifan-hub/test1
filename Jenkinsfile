pipeline {
    agent any
    
    environment {
        FEISHU_WEBHOOK = 'https://open.feishu.cn/open-apis/bot/v2/hook/b56f684a-9c78-4aec-b525-4d1a2e8998cc'
        FEISHU_SECRET_ID = 'c95c0f38-db1c-4175-b8ed-c17c32c5d65a'
    }
    
    stages {
        stage('Test Feishu Webhook') {
            steps {
                script {
                    // 1. 获取当前时间戳（整数秒）
                    def timestamp = (System.currentTimeMillis() / 1000) as Integer
                    echo "时间戳: ${timestamp}"
                    
                    // 2. 准备简单的测试消息
                    def payload = [
                        msg_type: "text",
                        content: [
                            text: "Jenkins测试消息 - 时间戳: ${timestamp}"
                        ]
                    ]
                    
                    def jsonPayload = groovy.json.JsonOutput.toJson(payload)
                    writeFile file: "${env.WORKSPACE}/test_payload.json", text: jsonPayload
                    
                    withCredentials([string(credentialsId: env.FEISHU_SECRET_ID, variable: 'SECRET')]) {
                        // 3. 精确按照飞书文档计算签名
                        sh """
                            #!/bin/bash
                            
                            echo "=== 飞书机器人签名测试 ==="
                            echo "Webhook: ${FEISHU_WEBHOOK}"
                            echo "Timestamp: ${timestamp}"
                            echo "Secret: \$SECRET"
                            
                            # 方法1：使用 echo -n（最常用）
                            echo "--- 方法1: echo -n ---"
                            SIGN_STRING="${timestamp}"'\n'  # 注意：这里是 '\\n' 字面量
                            SIGNATURE1=\$(echo -en "\${SIGN_STRING}\${SECRET}" | openssl dgst -sha256 -binary | base64)
                            echo "签名1: \$SIGNATURE1"
                            
                            # 方法2：使用 printf（更精确）
                            echo "--- 方法2: printf ---"
                            SIGNATURE2=\$(printf "%s\\n%s" "${timestamp}" "\$SECRET" | openssl dgst -sha256 -binary | base64)
                            echo "签名2: \$SIGNATURE2"
                            
                            # 方法3：直接拼接（官方示例风格）
                            echo "--- 方法3: 直接拼接 ---"
                            SIGNATURE3=\$(echo -n "${timestamp}"'\n'"\$SECRET" | openssl dgst -sha256 -binary | base64)
                            echo "签名3: \$SIGNATURE3"
                            
                            # 测试三种方法得到的签名是否一致
                            if [ "\$SIGNATURE1" = "\$SIGNATURE2" ] && [ "\$SIGNATURE2" = "\$SIGNATURE3" ]; then
                                echo "✓ 三种方法签名一致"
                                FINAL_SIGNATURE="\$SIGNATURE1"
                            else
                                echo "⚠ 签名不一致，使用方法1"
                                FINAL_SIGNATURE="\$SIGNATURE1"
                            fi
                            
                            echo "最终使用签名: \$FINAL_SIGNATURE"
                            
                            # 4. 发送测试请求（使用最简单的方式）
                            echo "发送请求到飞书..."
                            
                            # 方法A：使用文件
                            RESPONSE_A=\$(curl -s -w "\\nHTTP状态码: %{http_code}" \\
                                -X POST \\
                                -H 'Content-Type: application/json' \\
                                -H "X-Lark-Timestamp: ${timestamp}" \\
                                -H "X-Lark-Signature: \$FINAL_SIGNATURE" \\
                                -d @'${env.WORKSPACE}/test_payload.json' \\
                                '${FEISHU_WEBHOOK}')
                            
                            echo "响应A: \$RESPONSE_A"
                            
                            # 方法B：直接发送（备用）
                            echo "--- 备用方法：直接发送 ---"
                            RESPONSE_B=\$(curl -s \\
                                -X POST \\
                                -H 'Content-Type: application/json' \\
                                -H "X-Lark-Timestamp: ${timestamp}" \\
                                -H "X-Lark-Signature: \$FINAL_SIGNATURE" \\
                                -d '${jsonPayload.replace("'", "'\"'\"'")}' \\
                                '${FEISHU_WEBHOOK}')
                            
                            echo "响应B: \$RESPONSE_B"
                            
                            # 5. 显示原始请求信息（用于调试）
                            echo "=== 调试信息 ==="
                            echo "签名字符串: ${timestamp}"'\n'"\$SECRET"
                            echo "JSON Payload:"
                            cat '${env.WORKSPACE}/test_payload.json'
                            echo ""
                            
                            # 清理
                            rm -f '${env.WORKSPACE}/test_payload.json'
                        """
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "飞书通知测试完成"
        }
    }
}
