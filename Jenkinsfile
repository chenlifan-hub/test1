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
    }

}

