@Library('smartPipeline@main') _

pipeline {
    agent any
    

    options {
        timeout(time: 1, unit: 'HOURS')
    }stages {
        stage('Build Java API') {
            steps {
                smartBuild(type: 'maven', pom: 'backend/java-api/pom.xml', goals: 'test package')
            }
        }
        stage('Build AI') {
            steps {
                smartBuild(type: 'python', requirements: 'ai/linguatale-ai/requirements.txt')
            }
        }
        stage('Docker Build') {
            steps {
                smartContainer(image: "ghcr.io/rkstechforge/linguatale:${env.BUILD_NUMBER}", snykScan: false)
                sh 'docker tag ghcr.io/rkstechforge/linguatale:$BUILD_NUMBER ghcr.io/rkstechforge/linguatale:latest'
            }
        }
        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'GHCR_CREDENTIALS', usernameVariable: 'CR_USER', passwordVariable: 'CR_TOKEN')]) {
                    sh '''
                        echo "$CR_TOKEN" | docker login ghcr.io -u "$CR_USER" --password-stdin
                        docker push "ghcr.io/rkstechforge/linguatale:$BUILD_NUMBER"
                        docker push "ghcr.io/rkstechforge/linguatale:latest"
                    '''
                }
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'DEPLOY_SSH', keyFileVariable: 'KEY', usernameVariable: 'USER')]) {
                    sh 'ssh -i "$KEY" "$USER@$DEPLOY_HOST" "cd /opt/linguatale && docker compose up -d --build"'
                }
            }
        }
    }
}
