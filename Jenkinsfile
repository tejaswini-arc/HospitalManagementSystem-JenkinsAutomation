pipeline {

agent any

stages {

    stage('Detect Changes') {
        steps {
            script {

                def previousCommit = bat(
                    script: 'git rev-parse HEAD~1',
                    returnStdout: true
                ).trim()

                def currentCommit = bat(
                    script: 'git rev-parse HEAD',
                    returnStdout: true
                ).trim()

                def changedFiles = bat(
                    script: "git diff --name-only ${previousCommit} ${currentCommit}",
                    returnStdout: true
                ).trim()

                echo "Changed files:"
                echo changedFiles

                env.BACKEND_CHANGED = 'false'
                env.FRONTEND_CHANGED = 'false'

                def files = changedFiles.split('\r?\n')

                if (files.any { it.startsWith('HospitalManagement-Backend/') }) {
                    env.BACKEND_CHANGED = 'true'
                }

                if (files.any { it.startsWith('HospitalManagement-Frontend/') }) {
                    env.FRONTEND_CHANGED = 'true'
                }

                echo "Backend changed: ${env.BACKEND_CHANGED}"
                echo "Frontend changed: ${env.FRONTEND_CHANGED}"
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
