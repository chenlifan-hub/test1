pipeline {
    agent any

    environment {
        GITHUB_REPO = 'https://github.com/Hiveagents-ones/Hive.git' 
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                // 👇 直接获取 commit 信息
                sh 'echo "Commit message: $(git log -1 --pretty=%s)"'
            }
        }

        stage('Build') {
            steps {
                sh 'echo "正在编译..."'
                sh 'ls'
            }
        }

        // 👇 安全打印所有环境变量（可选）
        stage('Debug: Print Env Vars') {
            steps {
                sh 'printenv | sort'
            }
        }
    }
}
