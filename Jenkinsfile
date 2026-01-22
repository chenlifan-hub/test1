pipeline {
    agent any

    environment {
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

        // ✅ 安全方式：打印所有环境变量
        stage('Debug: Print Env Vars') {
            steps {
                script {
                    echo "=== 所有 Jenkins 环境变量 (安全方式) ==="
                    env.each { key, value ->
                        echo "${key} = ${value}"
                    }
                    echo "=== 打印完成 ==="
                }
            }
        }
    }
}
