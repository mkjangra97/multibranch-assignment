pipeline {
    agent any
    stages {
        stage('Prepare Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Code Checkout') {
            steps {
                checkout scm
                script {
                    // Most reliable branch detection
                    def rawBranch = env.GIT_LOCAL_BRANCH 
                        ?: env.GIT_BRANCH 
                        ?: env.BRANCH_NAME 
                        ?: 'unknown'
                    
                    // Remove "origin/" prefix if present
                    env.BRANCH = rawBranch.replaceAll('^origin/', '').trim()
                    
                    echo "Raw branch value: ${rawBranch}"
                    echo "Resolved branch: ${env.BRANCH}"
                }
            }
        }

        // ✅ Skip stage PEHLE aana chahiye
        stage('Skip Prefix Branch') {
            when {
                expression { env.BRANCH.startsWith('prefix') }
            }
            steps {
                script {
                    currentBuild.result = 'ABORTED'
                    error("prefix branches ke liye pipeline nahi chalegi. Branch: ${env.BRANCH}")
                }
            }
        }

        stage('Deploy to Production') {
            when {
                expression { env.BRANCH == 'main' }
            }
            steps {
                sh '''
                    echo "Deploying main branch to production server..."
                    sshpass -p 'm' rsync -avz -e 'ssh -o StrictHostKeyChecking=no' \
                        ./index.html manish@192.168.10.158:/var/www/main/
                '''
            }
        }

        stage('Deploy to Feature') {
            when {
                expression { env.BRANCH.startsWith('feature') }
            }
            steps {
                sh '''
                    echo "Deploying feature branch to feature server..."
                    sshpass -p 'm' rsync -avz -e 'ssh -o StrictHostKeyChecking=no' \
                        ./feature/index.html manish@192.168.10.158:/var/www/feature/
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline complete — branch: ${env.BRANCH}"
        }
        aborted {
            echo "⛔ Pipeline aborted — branch: ${env.BRANCH}"
        }
        failure {
            echo "❌ Pipeline failed — branch: ${env.BRANCH}"
        }
    }
}
