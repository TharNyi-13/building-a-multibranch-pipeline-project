pipeline {
    agent any
    
    tools {
        nodejs 'node21'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_TAG = ''
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'development', credentialsId: 'git-credentials', url: 'https://github.com/TharNyi-13/building-a-multibranch-pipeline-project.git'
            }
        }
        
        stage('Install Package Dependencies') {
            steps {
                sh "npm install"
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh "npm test"
            }
        }
        
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Vercelcron -Dsonar.projectName=Vercelcron"
                }
            }
        }
        
        stage('Determine Docker Tag') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'development') {
                        DOCKER_TAG = 'qat'
                    } else if (env.BRANCH_NAME == 'master') {
                        DOCKER_TAG = 'prod'
                    } else {
                        error "Unsupported branch: ${env.BRANCH_NAME}"
                    }
                }
            }
        }
        
        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
                        sh "docker build -t beyonddev/vercel-cron:$DOCKER_TAG ."
                    }
                }
            }
        }
        
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
                        sh "docker push beyonddev/vercel-cron:$DOCKER_TAG"
                    }
                }
            }
        }
        
        stage('Docker Deploy To Development') {
            when {
                branch 'development'
            }
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
                        sh "docker run -d -p 3000:3000 beyonddev/vercel-cron:$DOCKER_TAG"
                    }
                }
            }
        }
    }
}
