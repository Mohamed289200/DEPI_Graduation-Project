pipeline{
    agent any

    environment{
        IMAGE = '3laaharrrr/petclinic'
        VERSION = 'v2'
    }

    stages{
        // stage('Build jar file'){
        //     steps{
        //         script{
        //             echo 'building jar file.....'
        //             sh 'cd spring-petclinic; ./mvnw clean package -DskipTests'
        //             echo 'jar file built'
        //         }
        //     }
        // }
        // stage('Build Docker image') {
        //     steps{
        //         script{
        //             echo 'building image.....'
        //             sh "docker build -t ${IMAGE}:${VERSION} -t ${IMAGE}:latest ."
        //             echo 'image built'
        //         }
        //     }
        // }
        // stage('push image to dockerhub') {
        //     steps {
        //         script {
        //             echo "pushing image...."
        //             withCredentials([
        //                 usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')
        //              ]) {
        //                     sh "echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin"
        //                     sh "docker push ${IMAGE}:${VERSION}"
        //                     sh "docker push ${IMAGE}:latest"
        //                 }
        //             echo 'image pushed'    
        //         }
        //     }
        // }
        stage('Deploy to petclinic-server') {
            steps {
                script {
                    echo 'Deploying to petclinic-server...'
                    sshagent(['petclinic-serve']) {
                        sh "cd ansible-deploy; ansible-playbook -i inventory playbook.yaml"
                    }
                    echo 'Deployment completed'
                }
            }
        }        
        
    }

    // post {
    //     always {
    //         echo 'Cleaning up workspace...'
    //         cleanWs()
    //     }
    // }
}