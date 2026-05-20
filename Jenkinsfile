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
                sh 'docker compose build --no-cache'
            }
        }

        // ── 3. Stop old containers ────────────────────────────
        stage('Stop Old Containers') {
            steps {
                echo '🛑 Stopping old containers...'
                sh 'docker compose down --remove-orphans || true'
            }
        }

        // ── 4. Start new containers ───────────────────────────
        stage('Deploy to Localhost') {
            steps {
                echo '🚀 Starting updated containers...'
                sh 'docker compose up -d'
            }
        }

        // ── 5. Health check ───────────────────────────────────
        stage('Health Check') {
            steps {
                echo '✅ Checking if services are up...'
                // Wait 10 seconds for containers to start
                sh 'sleep 10'
                sh 'curl -f http://localhost:5000/api/health || exit 1'
                sh 'curl -f http://localhost:3000 || exit 1'
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
            sh 'docker compose logs --tail=50 || true'
        }
        always {
            echo "🏁 Pipeline finished — Build #${env.BUILD_NUMBER}"
        }
    }
}
