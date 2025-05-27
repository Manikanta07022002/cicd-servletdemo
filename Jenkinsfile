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
                git branch: 'master', url: 'https://github.com/Manikanta07022002/cicd-servlet.git'
            }
        }
        stage('Build with Maven') {
            steps {
                sh 'mvn clean install package'
            }
        }
        stage('Push to Nexus') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'loginpage', classifier: '', file: 'target/loginpage.war', type: 'war']], credentialsId: 'demo', groupId: 'com.login', nexusUrl: 'NEXUS_URL', nexusVersion: 'nexus3', protocol: 'http', repository: 'loginpage', version: '0.0.1-SNAPSHOT'
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
                    git "https://github.com/Manikanta07022002/cicd-servlet.git" 
                    sh "git config user.name $user"
                    sh "git config user.email veeramanikanta7020@gmail.com"
                    sh "sed -i 's+veera/loginpage.*+veera/loginpage:$params.dockertag+g' manifests/tomcat.yml"
                    sh "git add ."
                    sh "git commit -m Commit-$params.dockertag"
                    sh "git push https://$user:$passwd@github.com/$user/loginpage.git master"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubernetes-config']) {
                    sh 'kubectl apply -f manifests/'      
                    sh 'kubectl rollout restart deployment loginpage'
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
