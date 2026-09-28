
pipeline {

    agent any

    environment {
        DEPLOY_ENV = "/home/ubuntu/LeadFlow-Lead-Management-Application/.env"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out latest code...'

                checkout scm
            }
        }

        stage('Prepare Environment') {
            steps {
                sh '''
                    echo "Preparing environment file..."

                    if [ ! -f "$DEPLOY_ENV" ]; then
                        echo "ERROR: .env file not found at:"
                        echo "$DEPLOY_ENV"
                        exit 1
                    fi

                    cp "$DEPLOY_ENV" .env

                    echo ".env copied successfully."
                '''
            }
        }

        stage('Verify Files') {
            steps {
                sh '''
                    echo "Checking required deployment files..."

                    test -f docker-compose.yml
                    test -f nginx.conf
                    test -f Dockerfile.frontend
                    test -f backend/Dockerfile
                    test -f .env

                    echo "All required files are present."
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker images...'

                sh '''
                    docker compose build
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                echo 'Starting LeadFlow application...'

                sh '''
                    docker compose up -d
                '''
            }
        }

        stage('Container Status') {
            steps {
                echo 'Checking container status...'

                sh '''
                    sleep 10
                    docker compose ps
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo 'Checking LeadFlow API health...'

                sh '''
                    sleep 5
                    curl --fail http://localhost/api/health
                '''
            }
        }
    }

    post {

        success {
            echo '======================================'
            echo 'LeadFlow deployment successful!'
            echo '======================================'
        }

        failure {
            echo '======================================'
            echo 'LeadFlow deployment failed!'
            echo 'Showing container status and logs...'
            echo '======================================'

            sh '''
                docker compose ps || true
                docker compose logs --tail=100 || true
            '''
        }

        always {
            sh '''
                rm -f .env
            '''

            echo 'Pipeline execution completed.'
        }
    }
}


