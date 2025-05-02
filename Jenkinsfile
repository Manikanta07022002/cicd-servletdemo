pipeline {
    agent any
    tools {
        jdk 'JDK'
        maven 'Maven'
        git 'Git'
        dockerTool 'docker'
    }          
    environment {
        DOCKER_REGISTRY = 'your-registry'
        DOCKER_IMAGE = 'your-image'
        DOCKER_TAG = 'latest'
    }
    stages {
        stage('Clone Repository') {
            steps {
                cleanWs()
                git branch: 'master', url: 'https://github.com/dvsr1411/onlinebookstore.git'
            }
        }
        stage('Build with Maven') {
            steps {
                sh 'mvn clean install package'
            }
        }
        stage('Push to Nexus') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'onlinebookstore', classifier: '', file: 'target/onlinebookstore.war', type: 'war']], credentialsId: 'demo', groupId: 'onlinebookstore', nexusUrl: 'NEXUS_URL', nexusVersion: 'nexus3', protocol: 'http', repository: 'onlinebookstore', version: '0.0.1-SNAPSHOT'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://${DOCKER_REGISTRY}', 'docker-credentials-id') {
                        docker.image("${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                }
            }
        }
        stage('Update manifest') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'git', passwordVariable: 'passwd', usernameVariable: 'user')]) {
                    git "https://github.com/$user/onlinebookstore.git" 
                    sh "git config user.name $user"
                    sh "git config user.email satwikreddy.danda@hotmail.com"
                    sh "sed -i 's+dvsr1411/onlinebookstore.*+dvsr1411/onlinebookstore:$params.dockertag+g' manifests/tomcat.yml"
                    sh "git add ."
                    sh "git commit -m Commit-$params.dockertag"
                    sh "git push https://$user:$passwd@github.com/$user/onlinebookstore.git master"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubernetes-config']) {
                    sh 'kubectl apply -f manifests/'      
                    sh 'kubectl rollout restart deployment onlinebookstore'
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