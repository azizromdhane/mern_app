pipeline {
    agent any
    triggers { pollSCM('H/5 * * * *') } // vérifie tous les 5 min
    environment {
        IMAGE_SERVER = 'youruser/mern-server' // changez "youruser"
        IMAGE_CLIENT = 'youruser/mern-client' // changez "youruser"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'git@gitlab.com/yourgroup/yourrepo.git',
                    credentialsId: 'gitlab_ssh'
            }
        }

        stage('Build + Push SERVER') {
            when { changeset "server/**" } // ne build que si serveur modifié
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS'
                )]) {
                    sh '''
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        docker build -t $IMAGE_SERVER:${BUILD_NUMBER} server
                        docker push $IMAGE_SERVER:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Build + Push CLIENT') {
            when { changeset "client/**" } // ne build que si client modifié
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS'
                )]) {
                    sh '''
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        docker build -t $IMAGE_CLIENT:${BUILD_NUMBER} client
                        docker push $IMAGE_CLIENT:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Scan SERVER with Trivy') {
            when { changeset "server/**" }
            steps {
                sh '''
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                    aquasec/trivy image $IMAGE_SERVER:${BUILD_NUMBER} > trivy_report.txt || true
                    cat trivy_report.txt
                '''
            }
        }
    }

    post {
        always {
            sh 'docker system prune -af || true' // nettoyage
        }
    }
}
