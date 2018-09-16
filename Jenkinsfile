def remote = [:]
remote.name = 'dev'
remote.host = '127.0.0.1'
remote.user = 'jakub'
remote.allowAnyHosts = true

pipeline {
    environment {
        registryCredential = 'dockerhub'
    }
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Component tests') {
            steps {
                sh 'mvn clean package -Pcomponent-test'
            }
        }
        stage('Build Images') {
            steps {
                sh """
                    cd accounts
                    mvn dockerfile:build
                    cd ../authorization
                    mvn dockerfile:build
                    cd ../config
                    mvn dockerfile:build
                    cd ../exercises
                    mvn dockerfile:build
                    cd ../web
                    mvn dockerfile:build
                """
            }
        }
        stage('Push Images') {
            steps {
                sh """
                    cd accounts
                    mvn dockerfile:push
                    cd ../authorization
                    mvn dockerfile:push
                    cd ../config
                    mvn dockerfile:push
                    cd ../exercises
                    mvn dockerfile:push
                    cd ../web
                    mvn dockerfile:push
                """
            }
        }
        stage("Deploy") {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'dev', keyFileVariable: 'identity', passphraseVariable: 'passphrase', usernameVariable: 'userName')]) {
                    script {
                        remote.identity = identity
                        remote.user = userName
                        remote.passphrase = passphrase
                    }
                    sshScript remote: remote, script: "accounts/deploy/deploy.sh"
                }
            }
        }
    }
}