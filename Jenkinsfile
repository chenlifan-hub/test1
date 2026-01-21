pipeline {
    agent any

    environment {
        // 从 Git 提取仓库信息（适用于 Multibranch Pipeline）
        GITHUB_REPO = 'https://github.com/Hiveagents-ones/Hive.git' 
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'echo "正在编译..."'
                sh 'npm install && npm run build'
            }
        }
    }

    post {
        always {
            script {
                // 获取 commit SHA（Multibranch Pipeline 中可用 GIT_COMMIT）
                def commitSha = env.GIT_COMMIT ?: sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                def githubToken = credentials('a361e74a-84ef-45b3-9371-c370635149fa')

                def status = currentBuild.result ?: 'SUCCESS'
                def state = (status == 'SUCCESS') ? 'success' : 'failure'
                def description = (status == 'SUCCESS') ? '✅ Jenkins 编译成功' : '❌ Jenkins 编译失败'

                echo "向 GitHub 提交状态: ${state} for commit ${commitSha}"

                // 调用 GitHub Status API
                sh """
                    curl -s -X POST \\
                      -H "Authorization: token ${githubToken}" \\
                      -H "Accept: application/vnd.github.v3+json" \\
                      https://api.github.com/repos/${env.GITHUB_REPO}/statuses/${commitSha} \\
                      -d '{
                        "state": "${state}",
                        "target_url": "${env.BUILD_URL}",
                        "description": "${description}",
                        "context": "jenkins-ci"
                      }'
                """
            }
        }
    }
}

