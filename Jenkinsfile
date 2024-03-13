#!/usr/bin/env groovy

library identifier: 'jenkins-shared-library@master', retriever: modernSCM(
    [$class: 'GitSCMSource',
    remote: 'https://gitlab.com/twn-devops-bootcamp/latest/09-aws/jenkins-shared-library.git',
    credentialsID: 'gitlab-credentials'
    ]
)

pipeline {
    agent any
    tools {
        maven 'Maven-3.9'
    }
    environment {
        IMAGE_NAME = 'nirvanb/demo-app:jma-1.1'
    }
    stages {
        // stage('increment version') {
        //     steps {
        //         script {
        //             echo 'incrementing app version...'
        //             sh 'mvn build-helper:parse-version versions:set \
        //                 -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
        //                 versions:commit'
        //             def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
        //             def version = matcher[0][1]
        //             env.IMAGE_NAME = "$version-$BUILD_NUMBER"
        //         }
        //     }
        // }
        stage('build app') {
            steps {
                echo 'building application jar...'
                buildJar()
            }
        }
        stage('build image') {
            steps {
                script {
                    echo 'building the docker image...'
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        }
        stage("deploy") {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    // def dockerCmd = "docker run -p 8080:8080 -d ${IMAGE_NAME}"
                    def dockerComposeCmd = "docker-compose -f docker-compose.yaml up --detach"
                    def shellCmd = "bash ./server-cmds.sh"
                    sshagent(['ec2-server-key']) {
                        sh "scp server-cmds.sh ec2-user@34.239.0.149:/home/ec2-user"
                        sh "scp docker-compose.yaml ec2-user@34.239.0.149:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@34.239.0.149 ${shellCmd}"      
                    }
                }
            }
        } 
        // stage("deploy") {
        //     steps {
        //         script {
        //             echo 'deploying docker image to EC2...'

        //             def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
        //             def ec2Instance = "ec2-user@18.184.54.160"

        //             sshagent(['ec2-server-key']) {
        //                 sh "scp server-cmds.sh ${ec2Instance}:/home/ec2-user"
        //                 sh "scp docker-compose.yaml ${ec2Instance}:/home/ec2-user"
        //                 sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
        //             }
        //         }
        //     }               
        // }
        // stage('commit version update'){
        //     steps {
        //         script {
        //             withCredentials([usernamePassword(credentialsId: 'gitlab-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]){
        //                 sh 'git remote set-url origin https://$USER:$PASS@gitlab.com/twn-devops-bootcamp/latest/09-AWS/java-maven-app.git'
        //                 sh 'git add .'
        //                 sh 'git commit -m "ci: version bump"'
        //                 sh 'git push origin HEAD:jenkins-jobs'
        //             }
        //         }
        //     }
        // }
    }
}
