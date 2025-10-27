pipeline {
    agent any
    triggers { pollSCM('H/5 * * * *') } // Vérifie toutes les 5 minutes les changements
    environment {
        IMAGE_SERVER = 'azizromdhane/mern-server' // DockerHub
        IMAGE_CLIENT = 'azizromdhane/mern-client'
    }

    stages {
        stage('Checkout') {
            steps {
                git(
                    branch: 'main',
                    url: 'git@github.com:azizromdhane/mern_app.git',
                    credentialsId: 'github_ssh' // Clé SSH GitHub dans Jenkins
                )
            }
        }

        stage('Build + Push SERVER') {
            when {
                changeset pattern: 'server/**', comparator: 'ANT'
            }
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
            when {
                changeset pattern: 'client/**', comparator: 'ANT'
            }
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
            when {
                changeset pattern: 'server/**', comparator: 'ANT'
            }
            steps {
                sh '''
                    echo "🔍 Scanning image with Trivy..."
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                    aquasec/trivy image $IMAGE_SERVER:${BUILD_NUMBER} > trivy_report.txt || true

                    echo "=== Résumé du rapport Trivy ==="
                    cat trivy_report.txt | grep -E "CRITICAL|HIGH|MEDIUM" || true
                '''
            }
        }
    }

    post {
        always {
            echo "🧹 Nettoyage du système Docker..."
            sh 'docker system prune -af || true'
        }
    }
}
