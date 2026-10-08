pipeline {
    agent any

    environment {
        VIRTUAL_ENV = '.venv'
        DOCKER_IMAGE = 'slms-app'
        CONTAINER_NAME = 'slms-app'

        DJANGO_SETTINGS_MODULE = 'slms.settings'
        PYTHONPATH = "${WORKSPACE}/staffleave/slms:${WORKSPACE}/staffleave"
        WORKDIR = 'staffleave/slms'
        PYTEST = "${WORKSPACE}/.venv/bin/pytest"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Bilaalofficial/slms.git'

                sh '''
                    echo "Repository checked out"
                    echo "Workspace: $(pwd)"
                    ls -la
                '''
            }
        }

        stage('Set Up Python') {
            steps {
                sh '''
                    python3 -m venv "$VIRTUAL_ENV"

                    "$VIRTUAL_ENV/bin/pip" install --upgrade pip

                    "$VIRTUAL_ENV/bin/pip" install -r requirements.txt

                    "$VIRTUAL_ENV/bin/pip" install pytest pytest-django
                '''
            }
        }

        stage('Django Check') {
            steps {
                dir("${WORKDIR}") {
                    sh '''
                        "$WORKSPACE/.venv/bin/python" manage.py check
                    '''
                }
            }
        }

        stage('Run Tests') {
            steps {
                dir("${WORKDIR}") {
                    sh '''
                        "$WORKSPACE/.venv/bin/pytest" \
                            --ds=slms.settings \
                            --maxfail=1 \
                            --disable-warnings \
                            -v
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                        -t "$DOCKER_IMAGE:latest" \
                        .
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    echo "Stopping old SLMS container if it exists..."

                    docker rm -f "$CONTAINER_NAME" 2>/dev/null || true

                    echo "Starting new SLMS container..."

                    docker run -d \
                        --restart unless-stopped \
                        --name "$CONTAINER_NAME" \
                        -p 80:8000 \
                        "$DOCKER_IMAGE:latest"

                    echo "Container started:"
                    docker ps --filter "name=$CONTAINER_NAME"
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "Waiting for application..."
                    sleep 5

                    curl -f http://localhost/ || exit 1

                    echo ""
                    echo "SLMS deployment successful!"
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
            cleanWs()
        }
    }
}
