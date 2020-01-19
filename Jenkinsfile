pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
               echo "hello"  
               echo 'Running build automation'
                sh './mvnw install'
            }
        }


stage('DeployToStaging') {
            when {
                branch 'master'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'ssh-user1', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'user1',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ],
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'target/*.jar',
                                        removePrefix: 'target',
                                        remoteDirectory: '/home/user1/swjar/',
                                        execCommand: 'echo hello'
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }


 stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("sw3:5000/hello-test")
                    app.inside {
                        sh 'echo $(curl localhost:9282)'
                    }
                }
            }
        }


stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('http://sw3:5000', 'pvt-docker') {
                    app.push()
                    }
                }
            }
        }


stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'ssh-user1', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull sw3:5000/hello-test:latest\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop sw3:5000/hello-test\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm sw3:5000/hello-test\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name hello-test -p 9282:9282 -d sw3:5000/hello-test:latest\""
                    }
                }
            }
        }








}
}
