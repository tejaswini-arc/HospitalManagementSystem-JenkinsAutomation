pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/tejaswini-arc/HospitalManagementSystem-JenkinsAutomation.git'
            }
        }

        stage('Maven Build') {
            steps {
                dir('HospitalManagement-Backend') {
                    bat 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('HospitalManagement-Backend') {
                    bat 'docker build -t pixelbloom/hospitalmanagementsystem-backend:latest .'
                }
            }
        }

        stage('Check Trivy Installation') {
            steps {
                bat 'D:\\Software-Setups\\trivy_0.74.0\\trivy.exe --version'
            }
        }

        stage('Trivy Security Gate') {
            steps {
                bat '''
                    D:\\Software-Setups\\trivy_0.74.0\\trivy.exe image --severity HIGH,CRITICAL --exit-code 1 --no-progress pixelbloom/hospitalmanagementsystem-backend:latest
                '''
            }
        }

        stage('Docker Push') {
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
    }
}
