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
                    // Calculate branch AFTER checkout
                    env.BRANCH = env.GIT_LOCAL_BRANCH ?: env.GIT_BRANCH?.replaceAll('origin/', '') ?: 'unknown'
                    echo "Cloned branch: ${env.BRANCH}"
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
                        ./index.html manish@100.68.126.54/:/var/www/main/
                '''
            }
        }

        stage('Skip Branch') {
        when {
                expression { return env.BRANCH == 'prefix'}
            }
        steps {
            script {
            currentBuild.result = 'Aborted'
            error('prefix branches ke liye pipeline nahi Chalegi.')
            	    }
            }
        }

        stage('Deploy to Feature') {
            when {
                expression { env.BRANCH == 'feature' }
            }
            steps {
                sh '''
                    echo "Deploying feature branch to feature server..."
                    sshpass -p 'm' rsync -avz -e 'ssh -o StrictHostKeyChecking=no' \
                        ./feature/index.html manish@100.68.126.54:/var/www/feature/
                '''
            }
        }

    }

    post {
        success {
            echo "Pipeline complete — branch: ${env.BRANCH}"
        }
        failure {
            echo "Pipeline failed — branch: ${env.BRANCH}"
        }
    }
}
