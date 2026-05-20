pipeline {
    agent any

    environment {
        // Image names must match docker-compose.yml
        API_IMAGE  = 'student-management-system-api'
        APP_IMAGE  = 'student-management-system-app'
        TAG        = "build-${env.BUILD_NUMBER}"
    }

    stages {

        // ── 1. Pull latest code ───────────────────────────────
        stage('Checkout') {
            steps {
                echo '📥 Pulling latest code from GitHub...'
                checkout scm
            }
        }

        // ── 2. Build Docker images ────────────────────────────
        stage('Build Docker Images') {
            steps {
                echo '🐳 Building Docker images...'
                bat 'docker compose build --no-cache'
            }
        }

        // ── 3. Stop old containers ────────────────────────────
        stage('Stop Old Containers') {
            steps {
                echo '🛑 Stopping old containers...'
                bat 'docker compose down --remove-orphans || exit 0'
            }
        }

        // ── 4. Start new containers ───────────────────────────
        stage('Deploy to Localhost') {
            steps {
                echo '🚀 Starting updated containers...'
                bat 'docker compose up -d'
            }
        }

        // ── 5. Health check ───────────────────────────────────
        stage('Health Check') {
            steps {
                echo '✅ Checking if services are up...'
                // Wait 5 seconds for containers to start
                bat 'ping -n 6 127.0.0.1 > nul'
                bat 'curl -f http://localhost:5000/api/health || exit 1'
                bat 'curl -f http://localhost:3000 || exit 1'
                echo '✅ Both services are healthy!'
            }
        }
    }

    post {
        success {
            echo '''
            ╔══════════════════════════════════════╗
            ║  ✅ BUILD SUCCESS                    ║
            ║  Frontend: http://localhost:3000     ║
            ║  Backend:  http://localhost:5000/api ║
            ╚══════════════════════════════════════╝
            '''
        }
        failure {
            echo '❌ BUILD FAILED — check the logs above.'
            bat 'docker compose logs --tail=50 || exit 0'
        }
        always {
            echo "🏁 Pipeline finished — Build #${env.BUILD_NUMBER}"
        }
    }
}
