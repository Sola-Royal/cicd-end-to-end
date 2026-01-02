pipeline {
    
    agent any 
    
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {

        stage('Build Docker'){
            steps{
                script{
                    sh '''
                    echo 'Buid Docker Image'
                    docker build -t solaroyal/cicd-e2e:10:${BUILD_NUMBER} .
                    '''
                }
            }
        }

        stage('Push the artifacts') {
    steps {
        script {
            // This logs in using the credentials you just created
            withCredentials([usernamePassword(credentialsId: 'docker-hub-login', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                sh "echo $PASS | docker login -u $USER --password-stdin"
                sh "docker push solaroyal/cicd-e2e:10"
            }
        }
    }
}
        
        stage('Checkout K8S manifest SCM'){
            steps {
                git credentialsId: 'f87a34a8-0e09-45e7-b9cf-6dc68feac670', 
                url: 'https://github.com/Sola-Royal/cicd-demo-manifests-repo.git',
                branch: 'main'
            }
        }
        
        stage('Update K8S manifest & push to Repo'){
            steps {
                script{
                    withCredentials([usernamePassword(credentialsId: 'github-login', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh '''
                        cat deploy.yaml
                        // This command finds "image: <anything>" and replaces it with your specific image and build number
sh "sed -i 's|image: .*|image: solaroyal/cicd-e2e:${env.BUILD_NUMBER}|' deploy.yaml"
                        cat deploy.yaml
                        git add deploy.yaml
                        git commit -m 'Updated the deploy yaml | Jenkins Pipeline'
                        git remote -v
                        git push https://github.com/Sola-Royal/cicd-demo-manifests-repo.git HEAD:main
                        '''                        
                    }
                }
            }
        }
    }
}
