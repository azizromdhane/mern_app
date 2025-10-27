pipeline {
    agent any

    // Vérifie les changements sur GitHub toutes les 5 minutes
    triggers { pollSCM('H/5 * * * *') }

    environment {
        IMAGE_SERVER = 'azizromdhane/mern-server' // DockerHub
        IMAGE_CLIENT = 'azizromdhane/mern-client'
    }

    stages {

        stage('Checkout') {
            steps {
                echo "🔁 Clonage du dépôt GitHub..."
                git(
                    branch: 'main',
                    url: 'git@github.com:azizromdhane/mern_app.git',
                    credentialsId: 'github_ssh' // Doit exister dans Jenkins
                )
            }
        }

        stage('Build + Push SERVER') {
            when {
                changeset pattern: 'server/**', comparator: 'ANT'
            }
            steps {
                echo "⚙️ Construction et push de l'image du serveur..."
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
                echo "⚙️ Construction et push de l'image du client..."
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
                echo "🔍 Scan de l'image serveur avec Trivy..."
                sh '''
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
        success {
            echo "✅ Pipeline terminé avec succès !"
        }
        failure {
            echo "❌ Échec du pipeline — vérifier les logs."
        }
    }
}
