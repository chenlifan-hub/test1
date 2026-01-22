pipeline {
    agent any

    environment {
        FEISHU_WEBHOOK = 'https://open.feishu.cn/open-apis/bot/v2/hook/b56f684a-9c78-4aec-b525-4d1a2e8998cc'
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
            // ✅ 直接使用全限定类名，不要 import！
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

            // ✅ 关键：直接使用 groovy.json.JsonOutput.toJson()
            def jsonPayload = groovy.json.JsonOutput.toJson(payload)

            sh """
                curl -X POST \\
                  -H 'Content-Type: application/json' \\
                  -d '${jsonPayload}' \\
                  '${env.FEISHU_WEBHOOK}'
            """
        }
    }
}
}
