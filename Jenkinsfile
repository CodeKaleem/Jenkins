pipeline {
    agent any

    environment {
        DH_USER = 'kaleem830'
        TAG     = "${BUILD_NUMBER}"
        VM_IP   = '192.168.133.130'
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build Images') {
            steps {
                sh '''
                    docker build -t $DH_USER/product-backend:$TAG -t $DH_USER/product-backend:latest ./backend
                    docker build -t $DH_USER/product-frontend:$TAG -t $DH_USER/product-frontend:latest \
                        --build-arg NEXT_PUBLIC_API_URL=http://$VM_IP:5000/api ./frontend
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                        usernameVariable: 'U', passwordVariable: 'P')]) {
                    sh '''
                        echo "$P" | docker login -u "$U" --password-stdin
                        docker push $DH_USER/product-backend:$TAG
                        docker push $DH_USER/product-backend:latest
                        docker push $DH_USER/product-frontend:$TAG
                        docker push $DH_USER/product-frontend:latest
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker compose -p jenkins down || true
                    docker compose -p jenkins up -d
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    curl -f --retry 10 --retry-delay 3 --retry-connrefused http://localhost:5000/api/products
                    curl -f --retry 10 --retry-delay 3 --retry-connrefused -o /dev/null http://localhost:3000/
                    docker ps
                '''
            }
        }
    }

    post { always { sh 'docker logout || true' } }
}
