pipeline {
    environment {
        nexus = "13.50.194.17:5000"
        docker_image_name = "sprint-boot"
        BUILD_STATUS='FAILED'
        RECIPIENTS='shalitshlomo@gmail.com'
    }

    agent {
        label 'ubuntu'
    }

    triggers {
        pollSCM('* * * * *')
    }

    stages {
        stage('Checkout Code') {
            steps {
                cleanWs()
                checkout scmGit(branches: [[name: '*/main']], extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'app']], userRemoteConfigs: [[url: 'https://github.com/shlomoshalit123/react-java0mysql.git']])
                checkout scmGit(branches: [[name: '*/main']], extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'deploy']], userRemoteConfigs: [[url: 'https://github.com/shlomoshalit123/Spring-Boot-Deployment.git']])
            }
        }
        stage('build') {
            steps {
                sh 'docker compose -f app/docker-compose.yml up -d'
            }
        }
        stage('test') {
            steps {
                sh 'docker ps'
                sh 'sleep 30'
                sh 'curl 127.0.0.1:3000'
                sh 'echo "APPLICATION IS RUNNING"'
            }
        }
        stage('publish') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus', passwordVariable: 'nexus_password', usernameVariable: 'nexus_user')]) {
                    sh 'docker login -u ${nexus_user} -p ${nexus_password} http://${nexus}'
                    sh 'docker push ${nexus}/${docker_image_name}_frontend:${BUILD_NUMBER}'
                    sh 'docker push ${nexus}/${docker_image_name}_backend:${BUILD_NUMBER}'
                    sh 'docker push ${nexus}/${docker_image_name}_db:${BUILD_NUMBER}'
                }
            }
        }
    }
    post {
       always {
            sh 'echo ***** CLEANUP *****'    
            sh 'echo $(docker ps --filter="label=app=${docker_image_name}" --filter="label=build=${BUILD_NUMBER}")'
            sh 'docker rm -f $(docker ps --filter="label=app=${docker_image_name}" --filter="label=build=${BUILD_NUMBER}" -q)'
            sh 'docker rmi -f $(docker images --filter=reference="${nexus}/*:${BUILD_NUMBER}" -q)'
        }
    }
}
