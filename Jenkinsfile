#!groovy
@Library('jenkins-library@master')
import java.text.SimpleDateFormat

def dateFormat = new SimpleDateFormat("yyyy.MM.dd-HH.mm.ss")
def versionNumber = dateFormat.format(new Date())

def myAppTagVersion = params.versionNumber ?: versionNumber

pipeline {
    agent any

    parameters {
        string(defaultValue: "localhost:30500", description: 'Docker repository-Url, default: localhost:30500', name: 'RepoUrl')
        string(defaultValue: null, description: 'If left empty, the tag of myApp will be generated', name: 'myAppTagVersion')
        choice(choices: ['build', 'teardown'], description: 'Whether to build the service or to tear it down', name: 'deployAction')
    }

    stages {
        stage('Start') {
            steps {
                sendEMPSlackPENG("#439FE0","STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            }
        }
        stage('Docker build') {
            when {
                expression { "${params.deployAction}" ==~ /build/ }
            }
            steps {
                echo 'Building docker containers'
                script {
                    sh "docker build -f Dockerfile --pull -t ${RepoUrl}/my-app:${myAppTagVersion} ."
                }
            }
        }
        stage('Docker tag') {
            when {
                expression { "${params.deployAction}" ==~ /build/ }
            }
            steps {
                echo 'Tagging containers'
                script {
                    sh "docker tag ${RepoUrl}/my-app:${myAppTagVersion} ${RepoUrl}/my-app:latest"
                }
            }
        }
        stage('Docker Push') {
            when {
                expression { "${params.deployAction}" ==~ /build/ }
            }
            steps {
                echo 'Pushing Docker images to registry'
                script {
                    sh "docker push ${RepoUrl}/my-app:${myAppTagVersion}"
                    sh "docker push ${RepoUrl}/my-app:latest"
                }
            }
        }
        stage('Deploy my-app') {
            when {
                expression { "${params.deployAction}" ==~ /build/ }
            }
            steps {
                script {
                    sh "sed -i 's/<myAppTagVersion>/${myAppTagVersion}/g' infra/my-app/cronjob.yml"
                    sh "sed -i 's/<RepoUrl>/${RepoUrl}/g' infra/my-app/cronjob.yml"
                    sh "kubectl apply -f infra/my-app/ns.yml"
                    sh "kubectl apply -f infra/my-app/cronjob.yml"
                }
            }
        }
        stage('Delete my-app') {
            when {
                expression { "${params.deployAction}" ==~ /teardown/ }
            }
            steps {
                script {
                    sh "sed -i 's/<myAppTagVersion>/${myAppTagVersion}/g' infra/my-app/cronjob.yml"
                    sh "sed -i 's/<RepoUrl>/${RepoUrl}/g' infra/my-app/cronjob.yml"
                    sh "kubectl delete -f infra/my-app/cronjob.yml"
                }
            }
        }
    }
    post {
        success {
            sendEMPSlackPENG("good","SUCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        failure {
            sendEMPSlackPENG("danger","FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
    }
}
