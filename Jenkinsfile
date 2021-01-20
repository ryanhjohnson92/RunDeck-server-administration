pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/vagrant.zip'
            }
        }
        stage('DeployingToStaging'){
            when {
                branch 'master'
            }
            steps {
                withCredentials([usernamePassword(credentialId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')])
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'staging',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/vagrant.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/home/cloud_user/',
                                    )
                                ]
                            )
                        ]
                    )
            }
        }
        stage('DeployingToProduction'){
            when {
                branch 'master'
            }
            steps {
                input 'Does the staging environment look OK?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'production',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/vagrant.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/home/cloud_user',
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }
}