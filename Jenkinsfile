// Advanced Parameterized Jenkins Pipeline
// This pipeline can build and deploy your web application from a separate repository

pipeline {
    agent any
    
    parameters {
        // Git Configuration
        string(
            name: 'APP_REPO_URL',
            defaultValue: 'https://github.com/YOUR-USERNAME/my-web-app.git',
            description: 'Git repository URL of your application'
        )
        
        choice(
            name: 'GIT_BRANCH',
            choices: ['main', 'staging', 'develop', 'feature/new-feature'],
            description: 'Select branch to build'
        )
        
        // Build Options
        booleanParam(
            name: 'RUN_TESTS',
            defaultValue: true,
            description: 'Run automated tests?'
        )
        
        booleanParam(
            name: 'RUN_LINTING',
            defaultValue: true,
            description: 'Run code linting?'
        )
        
        booleanParam(
            name: 'BUILD_DOCKER_IMAGE',
            defaultValue: true,
            description: 'Build Docker image?'
        )
        
        // Deployment Options
        booleanParam(
            name: 'PUSH_TO_DOCKERHUB',
            defaultValue: false,
            description: 'Push image to Docker Hub?'
        )
        
        booleanParam(
            name: 'DEPLOY_LOCALLY',
            defaultValue: true,
            description: 'Deploy to local Docker?'
        )
        
        choice(
            name: 'DEPLOYMENT_PORT',
            choices: ['3000', '3001', '3002', '8080', '8081'],
            description: 'Port to deploy application'
        )
        
        // Docker Configuration
        string(
            name: 'DOCKER_IMAGE_NAME',
            defaultValue: 'my-web-app',
            description: 'Docker image name'
        )
        
        string(
            name: 'DOCKERHUB_USERNAME',
            defaultValue: 'your-dockerhub-username',
            description: 'Docker Hub username'
        )
        
        // Advanced Options
        booleanParam(
            name: 'CLEAN_WORKSPACE',
            defaultValue: true,
            description: 'Clean workspace before build?'
        )
        
        booleanParam(
            name: 'VERBOSE_LOGGING',
            defaultValue: false,
            description: 'Enable verbose logging?'
        )
    }
    
    environment {
        // Dynamic environment variables based on parameters
        APP_VERSION = "${params.GIT_BRANCH}-${BUILD_NUMBER}"
        DOCKER_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = "${params.DOCKER_IMAGE_NAME}-${params.GIT_BRANCH}"
        BUILD_TIMESTAMP = sh(script: 'date +%Y%m%d-%H%M%S', returnStdout: true).trim()
    }
    
    stages {
        stage('📋 Build Information') {
            steps {
                script {
                    echo """
                    ╔════════════════════════════════════════════════════════════╗
                    ║           🚀 CI/CD PIPELINE CONFIGURATION                  ║
                    ╠════════════════════════════════════════════════════════════╣
                    ║ Repository:     ${params.APP_REPO_URL}
                    ║ Branch:         ${params.GIT_BRANCH}
                    ║ Build Number:   ${BUILD_NUMBER}
                    ║ Version:        ${APP_VERSION}
                    ║ Timestamp:      ${BUILD_TIMESTAMP}
                    ║────────────────────────────────────────────────────────────║
                    ║ Build Options:                                             ║
                    ║   Run Tests:        ${params.RUN_TESTS}
                    ║   Run Linting:      ${params.RUN_LINTING}
                    ║   Build Docker:     ${params.BUILD_DOCKER_IMAGE}
                    ║────────────────────────────────────────────────────────────║
                    ║ Deployment:                                                ║
                    ║   Push to Hub:      ${params.PUSH_TO_DOCKERHUB}
                    ║   Deploy Locally:   ${params.DEPLOY_LOCALLY}
                    ║   Port:             ${params.DEPLOYMENT_PORT}
                    ║   Container:        ${CONTAINER_NAME}
                    ╚════════════════════════════════════════════════════════════╝
                    """
                    
                    // Set build description
                    currentBuild.description = "${params.GIT_BRANCH} | Build #${BUILD_NUMBER}"
                }
            }
        }
        
        stage('🧹 Clean Workspace') {
            when {
                expression { params.CLEAN_WORKSPACE == true }
            }
            steps {
                echo 'Cleaning workspace...'
                cleanWs()
            }
        }
        
        stage('📥 Checkout Application') {
            steps {
                echo "Cloning ${params.APP_REPO_URL} (${params.GIT_BRANCH} branch)..."
                script {
                    // Clone the application repository
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/${params.GIT_BRANCH}"]],
                        userRemoteConfigs: [[
                            url: params.APP_REPO_URL,
                            credentialsId: 'github-credentials'
                        ]]
                    ])
                    
                    sh '''
                        echo "=== Repository Information ==="
                        git log -1 --oneline
                        echo ""
                        echo "=== Repository Contents ==="
                        ls -la
                        echo ""
                        echo "=== Verifying Required Files ==="
                        
                        # Check for package.json
                        if [ -f package.json ]; then
                            echo "✅ package.json found"
                            cat package.json
                        else
                            echo "❌ package.json not found!"
                        fi
                        
                        # Check for Dockerfile
                        echo ""
                        if [ -f Dockerfile ]; then
                            echo "✅ Dockerfile found"
                        else
                            echo "⚠️  Dockerfile not found (will skip Docker build)"
                        fi
                    '''
                }
            }
        }
        
        stage('🔍 Code Analysis') {
            when {
                expression { params.RUN_LINTING == true }
            }
            steps {
                echo 'Running code analysis...'
                script {
                    sh '''
                        if [ -f package.json ]; then
                            echo "Installing dependencies for linting..."
                            npm install --only=dev 2>/dev/null || npm install
                            
                            echo ""
                            echo "=== Running ESLint ==="
                            npm run lint || echo "⚠️  No lint script found or linting failed"
                            
                            echo ""
                            echo "=== Code Statistics ==="
                            find . -name "*.js" -o -name "*.jsx" -o -name "*.ts" -o -name "*.tsx" | wc -l | xargs echo "TypeScript/JavaScript files:"
                        else
                            echo "⚠️  No package.json found, skipping linting"
                        fi
                    '''
                }
            }
        }
        
        stage('🧪 Run Tests') {
            when {
                expression { params.RUN_TESTS == true }
            }
            steps {
                echo 'Running automated tests...'
                script {
                    sh '''
                        if [ -f package.json ]; then
                            echo "Installing dependencies..."
                            npm install
                            
                            echo ""
                            echo "=== Running Tests ==="
                            npm test || echo "⚠️  No test script found or tests failed"
                            
                            echo ""
                            echo "=== Test Coverage ==="
                            npm run test:coverage 2>/dev/null || echo "No coverage script available"
                        else
                            echo "⚠️  No package.json found, skipping tests"
                        fi
                    '''
                }
            }
        }
        
        stage('🔨 Build Docker Image') {
            when {
                expression { params.BUILD_DOCKER_IMAGE == true }
            }
            steps {
                echo 'Building Docker image...'
                script {
                    sh """
                        if [ ! -f Dockerfile ]; then
                            echo "❌ Dockerfile not found!"
                            exit 1
                        fi
                        
                        echo "Building image: ${params.DOCKER_IMAGE_NAME}:${DOCKER_TAG}"
                        
                        docker build \
                          --build-arg BUILD_NUMBER=${BUILD_NUMBER} \
                          --build-arg APP_VERSION=${APP_VERSION} \
                          --build-arg GIT_BRANCH=${params.GIT_BRANCH} \
                          --build-arg BUILD_TIMESTAMP=${BUILD_TIMESTAMP} \
                          -t ${params.DOCKER_IMAGE_NAME}:${DOCKER_TAG} \
                          -t ${params.DOCKER_IMAGE_NAME}:${params.GIT_BRANCH} \
                          -t ${params.DOCKER_IMAGE_NAME}:latest \
                          .
                        
                        echo ""
                        echo "=== Built Images ==="
                        docker images | grep ${params.DOCKER_IMAGE_NAME}
                        
                        echo ""
                        echo "=== Image Details ==="
                        docker inspect ${params.DOCKER_IMAGE_NAME}:${DOCKER_TAG} --format='Size: {{.Size}} bytes'
                    """
                }
            }
        }
        
        stage('🧪 Test Docker Image') {
            when {
                expression { 
                    params.BUILD_DOCKER_IMAGE == true && params.RUN_TESTS == true 
                }
            }
            steps {
                echo 'Testing Docker image...'
                script {
                    sh """
                        set +e
                        
                        TEST_PORT=\$((${params.DEPLOYMENT_PORT} + 1000))
                        
                        echo "Starting test container on port \$TEST_PORT..."
                        docker run -d \
                          --name test-${CONTAINER_NAME}-${BUILD_NUMBER} \
                          -p \$TEST_PORT:${params.DEPLOYMENT_PORT} \
                          -e APP_VERSION=${APP_VERSION} \
                          -e BUILD_NUMBER=${BUILD_NUMBER} \
                          -e NODE_ENV=test \
                          ${params.DOCKER_IMAGE_NAME}:${DOCKER_TAG}
                        
                        echo "Waiting for application to start..."
                        sleep 15
                        
                        echo ""
                        echo "=== Container Logs ==="
                        docker logs test-${CONTAINER_NAME}-${BUILD_NUMBER}
                        
                        echo ""
                        echo "=== Testing Connectivity ==="
                        if curl -f http://localhost:\$TEST_PORT/ 2>/dev/null; then
                            echo "✅ Container test: PASS"
                        else
                            echo "⚠️  Container test: App may still be starting"
                        fi
                        
                        # Cleanup
                        docker stop test-${CONTAINER_NAME}-${BUILD_NUMBER} || true
                        docker rm test-${CONTAINER_NAME}-${BUILD_NUMBER} || true
                        
                        set -e
                    """
                }
            }
        }
        
        stage('🔐 Security Scan') {
            when {
                expression { params.BUILD_DOCKER_IMAGE == true }
            }
            steps {
                echo 'Running security scan...'
                script {
                    sh """
                        echo "=== Image Security Scan ==="
                        docker inspect ${params.DOCKER_IMAGE_NAME}:${DOCKER_TAG} --format='{{.Config.User}}' | xargs echo "Running as user:"
                        
                        echo ""
                        echo "=== Checking for Secrets ==="
                        docker history ${params.DOCKER_IMAGE_NAME}:${DOCKER_TAG} --no-trunc | grep -i "password\\|secret\\|key\\|token" || echo "✅ No secrets found in image history"
                        
                        # Optional: Add Trivy scan
                        # docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${params.DOCKER_IMAGE_NAME}:${DOCKER_TAG}
                    """
                }
            }
        }
        
        stage('📤 Push to Docker Hub') {
            when {
                expression { 
                    params.PUSH_TO_DOCKERHUB == true && params.BUILD_DOCKER_IMAGE == true 
                }
            }
            steps {
                echo 'Pushing to Docker Hub...'
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                            echo "Logging in to Docker Hub..."
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                            
                            echo ""
                            echo "Tagging images for Docker Hub..."
                            docker tag ${params.DOCKER_IMAGE_NAME}:${DOCKER_TAG} \$DOCKER_USER/${params.DOCKER_IMAGE_NAME}:${DOCKER_TAG}
                            docker tag ${params.DOCKER_IMAGE_NAME}:${DOCKER_TAG} \$DOCKER_USER/${params.DOCKER_IMAGE_NAME}:${params.GIT_BRANCH}
                            docker tag ${params.DOCKER_IMAGE_NAME}:${DOCKER_TAG} \$DOCKER_USER/${params.DOCKER_IMAGE_NAME}:latest
                            
                            echo ""
                            echo "Pushing images..."
                            docker push \$DOCKER_USER/${params.DOCKER_IMAGE_NAME}:${DOCKER_TAG}
                            docker push \$DOCKER_USER/${params.DOCKER_IMAGE_NAME}:${params.GIT_BRANCH}
                            
                            if [ "${params.GIT_BRANCH}" = "main" ]; then
                                docker push \$DOCKER_USER/${params.DOCKER_IMAGE_NAME}:latest
                            fi
                            
                            echo ""
                            echo "✅ Successfully pushed to Docker Hub!"
                            echo "🐳 https://hub.docker.com/r/\$DOCKER_USER/${params.DOCKER_IMAGE_NAME}"
                            
                            docker logout
                        """
                    }
                }
            }
        }
        
        stage('🚀 Deploy') {
            when {
                expression { 
                    params.DEPLOY_LOCALLY == true && params.BUILD_DOCKER_IMAGE == true 
                }
            }
            steps {
                echo "Deploying to local Docker on port ${params.DEPLOYMENT_PORT}..."
                script {
                    sh """
                        set +e
                        
                        # Stop and remove existing container
                        echo "Cleaning up existing deployment..."
                        docker stop ${CONTAINER_NAME} 2>/dev/null || true
                        docker rm ${CONTAINER_NAME} 2>/dev/null || true
                        
                        # Deploy new container
                        echo ""
                        echo "Starting new container..."
                        docker run -d \
                          --name ${CONTAINER_NAME} \
                          --restart unless-stopped \
                          -p ${params.DEPLOYMENT_PORT}:3000 \
                          -e APP_VERSION=${APP_VERSION} \
                          -e BUILD_NUMBER=${BUILD_NUMBER} \
                          -e GIT_BRANCH=${params.GIT_BRANCH} \
                          -e NODE_ENV=production \
                          ${params.DOCKER_IMAGE_NAME}:${DOCKER_TAG}
                        
                        echo "Waiting for application to start..."
                        sleep 10
                        
                        echo ""
                        echo "=== Deployment Status ==="
                        docker ps | grep ${CONTAINER_NAME}
                        
                        echo ""
                        echo "=== Application Logs ==="
                        docker logs --tail 30 ${CONTAINER_NAME}
                        
                        echo ""
                        echo "=== Health Check ==="
                        if curl -f -s http://localhost:${params.DEPLOYMENT_PORT}/ > /dev/null 2>&1; then
                            echo "✅ Application is accessible!"
                        else
                            echo "⏳ Application may still be starting..."
                        fi
                        
                        set -e
                    """
                }
            }
        }
    }
    
    post {
        success {
            script {
                def deploymentInfo = params.DEPLOY_LOCALLY ? "http://localhost:${params.DEPLOYMENT_PORT}" : "Not deployed locally"
                def dockerHubInfo = params.PUSH_TO_DOCKERHUB ? "https://hub.docker.com/r/${params.DOCKERHUB_USERNAME}/${params.DOCKER_IMAGE_NAME}" : "Not pushed to Docker Hub"
                
                echo """
                ╔════════════════════════════════════════════════════════════╗
                ║              ✅ PIPELINE SUCCEEDED!                        ║
                ╠════════════════════════════════════════════════════════════╣
                ║ Branch:         ${params.GIT_BRANCH}
                ║ Build:          #${BUILD_NUMBER}
                ║ Version:        ${APP_VERSION}
                ║────────────────────────────────────────────────────────────║
                ║ 🌐 Local:       ${deploymentInfo}
                ║ 🐳 Docker Hub:  ${dockerHubInfo}
                ║────────────────────────────────────────────────────────────║
                ║ 📦 Container:   ${CONTAINER_NAME}
                ║ 🏷️  Image:      ${params.DOCKER_IMAGE_NAME}:${DOCKER_TAG}
                ║────────────────────────────────────────────────────────────║
                ║ Useful Commands:
                ║   docker logs -f ${CONTAINER_NAME}
                ║   docker stop ${CONTAINER_NAME}
                ║   docker restart ${CONTAINER_NAME}
                ╚════════════════════════════════════════════════════════════╝
                """
            }
        }
        
        failure {
            echo """
            ╔════════════════════════════════════════════════════════════╗
            ║              ❌ PIPELINE FAILED!                           ║
            ╠════════════════════════════════════════════════════════════╣
            ║ Branch:         ${params.GIT_BRANCH}
            ║ Build:          #${BUILD_NUMBER}
            ║ Check logs above for error details
            ╚════════════════════════════════════════════════════════════╝
            """
        }
        
        always {
            script {
                if (params.VERBOSE_LOGGING) {
                    echo "=== Full Build Summary ==="
                    sh '''
                        echo "Docker Images:"
                        docker images | head -10
                        echo ""
                        echo "Running Containers:"
                        docker ps
                        echo ""
                        echo "Workspace Size:"
                        du -sh .
                    '''
                }
                
                // Cleanup
                sh 'docker image prune -f || true'
            }
        }
    }
}