def clientIP = ""
def acmeMode = "prod"

pipeline {
    agent {
        label 'pi'
    }
    environment {
        API_USER = credentials('namecheap-username')
        API_KEY = credentials('namecheap-api-key')
        USERNAME = credentials('namecheap-username')
        SLD = 'haomingyin'
        TLD = 'com'
        APPLY_DOMAIN = '*.haomingyin.com'
        SCRIPTS_REPO = 'https://github.com/haomingyin/certbot-namecheap-hook.git'
        EMAIL = credentials('admin-email')
    }
    triggers {
        cron('H H * * H')
    }
    stages {
        stage('Pull Script') {
            when {
                not {
                    branch 'PR-*'
                }
            }
            steps {
                sh "mkdir scripts"
                sh "git clone ${env.SCRIPTS_REPO} ./scripts"
            }
        }

        stage('Run Staging?') {
            when {
                not {
                    branch 'PR-*'
                }
            } 
            steps {
                script {
                    try {
                        timeout(time: 5, unit: 'MINUTES') {
                            acmeMode = input message: "Which environment do you want to apply", ok: 'Proceed',
                                parameters: [
                                    choice(name: 'acmeMode', choices: 'prod\nstaging', description: 'Production mode or staging mode')
                                ]
                        }
                        echo "Input received. Running for '${acmeMode}' mode"
                    } catch (err) {
                        echo "Timeout reached. Running for the default mode: '${acmeMode}'"
                    }
                }
            }
        }

        stage('Get Client IP') {
            when {
                not {
                    branch 'PR-*'
                }
            }
            steps {
                script {
                    dir('scripts') {
                        sh "./utility.sh get_client_ip > .ip.tmp"
                        clientIP = readFile(".ip.tmp")
                        echo "Client IP is: ${clientIP}"
                    }
                }
            }
        }

        stage('Obtain Cert - Staging') {
            when {
                allOf {
                    not {
                        branch 'PR-*'
                    }
                    expression {
                        "${acmeMode}" == 'staging'
                    }
                }
                
            }
            environment {
                ACME_SERVER="https://acme-staging-v02.api.letsencrypt.org/directory"
                CLIENT_IP = "${clientIP}"
            }
            steps {
                script {
                   dir('scripts') {
                        sh "PATH=$PATH:/usr/local/bin certbot certonly \
                        --manual \
                        --logs-dir ./var/log/letsencrypt \
                        --work-dir ./var/letsencrypt \
                        --config-dir ./etc/letsencrypt \
                        --preferred-challenges=dns \
                        --manual-auth-hook ./authenticator.sh \
                        --manual-cleanup-hook ./cleanup.sh \
                        -d ${env.APPLY_DOMAIN} \
                        -m ${env.EMAIL} \
                        --server ${env.ACME_SERVER} \
                        --agree-tos \
                        --manual-public-ip-logging-ok \
                        --force-renewal \
                        --break-my-certs"
                    }
                }
            }
        }

        stage('Obtain Cert - Prod') {
            when {
                allOf {
                    not {
                        branch 'PR-*'
                    }
                    expression {
                        "${acmeMode}" == 'prod'
                    }               
                }
            }
            environment {
                ACME_SERVER = "https://acme-v02.api.letsencrypt.org/directory"
                CLIENT_IP = "${clientIP}"
            }
            steps {
                script {
                    dir('scripts') {
                        sh "PATH=$PATH:/usr/local/bin certbot certonly \
                        --manual \
                        --logs-dir /var/log/letsencrypt \
                        --work-dir /var/letsencrypt \
                        --config-dir /etc/letsencrypt \
                        --preferred-challenges=dns \
                        --manual-auth-hook ./authenticator.sh \
                        --manual-cleanup-hook ./cleanup.sh \
                        -d ${env.APPLY_DOMAIN} \
                        -m ${env.EMAIL} \
                        --server ${env.ACME_SERVER} \
                        --agree-tos \
                        --manual-public-ip-logging-ok \
                        --force-renewal"
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
