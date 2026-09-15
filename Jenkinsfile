pipeline {

    agent any

    options {
        skipDefaultCheckout(true)
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Detect Changes') {
            steps {
                script {

                    env.BACKEND_CHANGED = 'false'
                    env.FRONTEND_CHANGED = 'false'

                    echo "Checking files changed in this build..."

                    for (changeLogSet in currentBuild.changeSets) {

                        for (entry in changeLogSet.items) {

                            echo "Commit: ${entry.commitId}"
                            echo "Message: ${entry.msg}"

                            for (file in entry.affectedFiles) {

                                echo "Changed file: ${file.path}"

                                if (file.path.startsWith('HospitalManagement-Backend/')) {
                                    env.BACKEND_CHANGED = 'true'
                                }

                                if (file.path.startsWith('HospitalManagement-Frontend/')) {
                                    env.FRONTEND_CHANGED = 'true'
                                }
                            }
                        }
                    }

                    echo "===================================="
                    echo "Backend changed  : ${env.BACKEND_CHANGED}"
                    echo "Frontend changed : ${env.FRONTEND_CHANGED}"
                    echo "===================================="
                }
            }
        }

        stage('Backend Maven Build') {
            when {
                expression {
                    env.BACKEND_CHANGED == 'true'
                }
            }

            steps {
                dir('HospitalManagement-Backend') {
                    bat 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Backend Docker Build') {
            when {
                expression {
                    env.BACKEND_CHANGED == 'true'
                }
            }

            steps {
                dir('HospitalManagement-Backend') {
                    bat 'docker build -t pixelbloom/hospitalmanagementsystem-backend:latest .'
                }
            }
        }

        stage('Backend Trivy Security Gate') {
            when {
                expression {
                    env.BACKEND_CHANGED == 'true'
                }
            }

            steps {
                bat '''
                    D:\\Software-Setups\\trivy_0.74.0\\trivy.exe image --severity HIGH,CRITICAL --exit-code 1 --no-progress pixelbloom/hospitalmanagementsystem-backend:latest
                '''
            }
        }

        stage('Backend Docker Push') {
            when {
                expression {
                    env.BACKEND_CHANGED == 'true'
                }
            }

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentialss',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    bat 'echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin'

                    bat 'docker push pixelbloom/hospitalmanagementsystem-backend:latest'
                }
            }
        }

        stage('Frontend Docker Build') {
            when {
                expression {
                    env.FRONTEND_CHANGED == 'true'
                }
            }

            steps {
                dir('HospitalManagement-Frontend') {
                    bat 'docker build -t pixelbloom/hospitalmanagementsystem-frontend:latest .'
                }
            }
        }

        stage('Frontend Trivy Security Gate') {
            when {
                expression {
                    env.FRONTEND_CHANGED == 'true'
                }
            }

            steps {
                bat '''
                    D:\\Software-Setups\\trivy_0.74.0\\trivy.exe image --severity HIGH,CRITICAL --exit-code 1 --no-progress pixelbloom/hospitalmanagementsystem-frontend:latest
                '''
            }
        }

        stage('Frontend Docker Push') {
            when {
                expression {
                    env.FRONTEND_CHANGED == 'true'
                }
            }

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentialss',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    bat 'echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin'

                    bat 'docker push pixelbloom/hospitalmanagementsystem-frontend:latest'
                }
            }
        }
    }

    post {

        success {
            echo 'CI/CD pipeline completed successfully.'
        }

        failure {
            echo 'CI/CD pipeline failed. Check the stage logs above.'
        }
    }
}