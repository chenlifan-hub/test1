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
                sh 'ls'
            }
        }

        // 👇 新增：打印所有环境变量（用于调试）
        stage('Debug: Print All Env Vars') {
            steps {
                script {
                    echo "=== 所有 Jenkins 环境变量 ==="
                    env.getEnvironment().each { key, value ->
                        echo "${key} = ${value}"
                    }
                    echo "=== 环境变量打印完成 ==="
                }
            }
        }
    }
}
