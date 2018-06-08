pipeline {
    agent {
        label 'master'
    }
    environment {
        API_USER = 'haomingyin'
        API_KEY = credentials('namecheap-api-key')
        USERNAME = 'haomingyin'
        SLD = 'haomingyin'
        TLD = 'com'
        SCRIPTS_REPO = 'https://github.com/haomingyin/certbot-namecheap-hook.git'
    }
    triggers {
        cron('H H * * H')
    }
    stages {
        stage('Pull Script') {
            when {
                branch {
                    not 'PR-'
                }
            }
            steps {
                script {
                    sh "mkdir scripts"
                    sh "git clone ${env.SCRIPTS_REPO} ./scripts"
                }
            }
        }

        stage('Obtain Cert') {
            when {
                branch {
                    not 'PR-'
                }
            }
            steps {
                script {
                    dir('scripts') {
                        sh "./main.sh"
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}