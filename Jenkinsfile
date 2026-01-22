pipeline {
    agent any
    
    environment {
        FEISHU_WEBHOOK = 'https://open.feishu.cn/open-apis/bot/v2/hook/b56f684a-9c78-4aec-b525-4d1a2e8998cc'
        FEISHU_SECRET_ID = 'c95c0f38-db1c-4175-b8ed-c17c32c5d65a'
    }
    
    stages {
        stage('Test with Python') {
            steps {
                script {
                    def timestamp = (System.currentTimeMillis() / 1000) as Integer
                    
                    withCredentials([string(credentialsId: env.FEISHU_SECRET_ID, variable: 'SECRET')]) {
                        // 使用 Python 计算签名（最准确）
                        sh """
                            #!/bin/bash
                            
                            echo "使用 Python 计算签名..."
                            
                            cat > ${env.WORKSPACE}/feishu_sign.py << 'EOF'
import hmac
import hashlib
import base64
import time
import sys
import os

timestamp = "${timestamp}"
secret = os.environ.get('SECRET')

# 构造签名字符串
string_to_sign = f'{timestamp}\\n{secret}'

# 计算 HMAC-SHA256
hmac_code = hmac.new(
    secret.encode('utf-8'), 
    string_to_sign.encode('utf-8'), 
    digestmod=hashlib.sha256
).digest()

# Base64 编码
signature = base64.b64encode(hmac_code).decode('utf-8')

print(f"Timestamp: {timestamp}")
print(f"Secret: {secret}")
print(f"String to sign: {repr(string_to_sign)}")
print(f"Signature: {signature}")

# 写入环境变量供后续使用
with open('${env.WORKSPACE}/signature.txt', 'w') as f:
    f.write(signature)
EOF
                            
                            python3 ${env.WORKSPACE}/feishu_sign.py
                            
                            SIGNATURE=\$(cat ${env.WORKSPACE}/signature.txt)
                            
                            # 发送测试请求
                            curl -X POST \\
                                -H 'Content-Type: application/json' \\
                                -H "X-Lark-Timestamp: ${timestamp}" \\
                                -H "X-Lark-Signature: \$SIGNATURE" \\
                                -d '{"msg_type":"text","content":{"text":"Python签名测试成功！时间戳: ${timestamp}"}}' \\
                                '${FEISHU_WEBHOOK}'
                                
                            # 清理
                            rm -f ${env.WORKSPACE}/feishu_sign.py ${env.WORKSPACE}/signature.txt
                        """
                    }
                }
            }
        }
    }
}
