pipeline{
    agent any

    environment{
        IMAGE = '3laaharrrr/petclinic'
        VERSION = 'v2'
        EMAIL = '3laahany946@gmail.com'
        SLACK_CHANNEL = 'jenkins'
    }

    stages{
        stage('Build jar file'){
            steps{
                script{
                    echo 'building jar file.....'
                    sh 'cd spring-petclinic; ./mvnw clean package -DskipTests'
                    echo 'jar file built'
                }
            }
        }
        stage('Build Docker image') {
            steps{
                script{
                    echo 'building image.....'
                    sh "docker build -t ${IMAGE}:${VERSION} -t ${IMAGE}:latest ."
                    echo 'image built'
                }
            }
        }
        stage('push image to dockerhub') {
            steps {
                script {
                    echo "pushing image...."
                    withCredentials([
                        usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')
                     ]) {
                            sh "echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin"
                            sh "docker push ${IMAGE}:${VERSION}"
                            sh "docker push ${IMAGE}:latest"
                        }
                    echo 'image pushed'    
                }
            }
        }
        stage('Deploy to petclinic-server') {
            steps {
                script {
                    echo 'Deploying to petclinic-server...'
                    sshagent(['petclinic-server']) {
                        sh "cd ansible-deploy; ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory playbook.yaml"
                    }
                    echo 'Deployment completed'
                }
            }
        }        
        
    }

    post {
        success {
            emailext (
                to: "${EMAIL}",
                from: "jenkins@test.com",
                subject: "Deployment Successful: Jenkins Build ${currentBuild.fullDisplayName}",
                body: """The deployment to the petclinic-server was successful.
                
                Job URL: ${env.BUILD_URL}
                """
            )

            slackSend (
                channel: SLACK_CHANNEL,
                color: 'good',
                message: "Build Success: ${env.JOB_NAME} - ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            )

        }
        failure {
            emailext (
                to: "${EMAIL}",
                from: "jenkins@test.com",
                subject: "Deployment Failed: Jenkins Build ${currentBuild.fullDisplayName}",
                body: """The deployment to the petclinic-server failed.
                
                Job URL: ${env.BUILD_URL}
                """
            )

            slackSend (
                channel: SLACK_CHANNEL,
                color: 'danger',
                message: "Build Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            )
        }
    }
}