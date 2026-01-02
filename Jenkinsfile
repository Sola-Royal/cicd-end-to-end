pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Build Docker'){
            steps{
                script{
                    // Use double quotes so ${env.BUILD_NUMBER} works
                    sh """
                        echo 'Build Docker Image'
                        docker build -t solaroyal/cicd-e2e:${env.BUILD_NUMBER} .
                    """
                }
            }
        }

        stage('Push the artifacts') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-login', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push solaroyal/cicd-e2e:${env.BUILD_NUMBER}"
                    }
                }
            }
        }

        stage('Checkout K8S manifest SCM'){
            steps {
                // IMPORTANT: Changed to 'github-login' (Your ID) instead of the old broken one
                git credentialsId: 'github-login', 
                url: 'https://github.com/Sola-Royal/cicd-demo-manifests-repo.git',
                branch: 'main'
            }
        }

        stage('Update K8S manifest & push to Repo'){
            steps {
                script{
                    withCredentials([usernamePassword(credentialsId: 'github-login', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        // Use Triple Double Quotes (""") for multi-line shell scripts with variables
                        sh """
                            cat deploy.yaml
                            
                            # Replace the image tag (Use # for comments in shell, not //)
                            sed -i 's|image: .*|image: solaroyal/cicd-e2e:${env.BUILD_NUMBER}|' deploy.yaml
                            
                            cat deploy.yaml
                            
                            git config user.email "jenkins@example.com"
                            git config user.name "Jenkins"
                            
                            git add deploy.yaml
                            git commit -m 'Updated the deploy yaml | Jenkins Pipeline'
                            
                            # Use the credentials to push securely
                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Sola-Royal/cicd-demo-manifests-repo.git HEAD:main
                        """
                    }
                }
            }
        }
    }
}
