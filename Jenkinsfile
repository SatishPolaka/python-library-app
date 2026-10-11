pipeline {
    agent {
        node {
            label 'prod'
        }
    }

    stages {
        stage('CODE') {
            steps {
                git 'https://github.com/SatishPolaka/python-library-app.git'
            }
        }

        stage('SCQ') {
            steps {
                script {
                    def scannerHome = tool 'sonar'
                    withSonarQubeEnv('sonar') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=python-library-app \
                            -Dsonar.sources=.
                        """
                    }
                }
            }
        }
        
        stage ('QUALITY GATES') {
            steps {
                waitForQualityGate abortPipeline: true, credentialsId: 'admin'
            }
        }
        
        stage ('IMAGES') {
            steps {
                sh 'docker build -t auth-service ./auth'
                sh 'docker build -t book-service ./book'
                sh 'docker build -t borrow-service ./borrow'
                sh 'docker build -t dbimage ./database'
                sh 'docker build -t python-app .'
            }
        }
        
        stage ('IMAGE SCAN') {
            steps {
                sh 'trivy image auth-service >> auth-image-trivyreport.txt'
                sh 'trivy image book-service >> book-image-trivyreport.txt'
                sh 'trivy image borrow-service >> borrow-image-trivyreport.txt'
                sh 'trivy image dbimage >> db-image-trivyreport.txt'
                sh 'trivy image python-app >> python-app-trivyreport.txt'
            }
        }
        
        stage ('REGISTORY') {
            steps {
                withDockerRegistry(credentialsId: 'docker password') {
                    sh 'docker tag auth-service satishpolaka/auth-service:v1'
                    sh 'docker tag book-service satishpolaka/book-service:v1'
                    sh 'docker tag borrow-service satishpolaka/borrow-service:v1'
                    sh 'docker tag dbimage satishpolaka/dbimage:v1'
                    sh 'docker tag python-app satishpolaka/python-app:v1'
                    
                    sh 'docker push satishpolaka/auth-service:v1'
                    sh 'docker push satishpolaka/book-service:v1'
                    sh 'docker push satishpolaka/borrow-service:v1'
                    sh 'docker push satishpolaka/dbimage:v1'
                    sh 'docker push satishpolaka/python-app:v1'
                }
            }
        }
        
        stage ('DEPLOY') {
            steps {
                sh 'docker stack deploy myapp --compose-file=compose.yml'
            }
        }
    }
}
