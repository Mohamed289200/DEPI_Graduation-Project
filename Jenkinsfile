pipeline {
    agent any

    environment {
        IMAGE = '3laaharrrr/petclinic'
        VERSION = 'v2'
        EMAIL = '3laahany946@gmail.com'
        SLACK_CHANNEL = 'jenkins'
        IGNORE_COMMIT_MESSAGE = '[skip ci]' // Commit message to ignore CI builds
    }

    stages {
        // stage('Increment Version') {
        //     steps {
        //         script {
        //             // Increment the version in pom.xml
        //             sh """
        //             cd spring-petclinic; \
        //             mvn build-helper:parse-version versions:set \
        //             -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
        //             versions:commit 
        //             """
        //             def temp = readFile('spring-petclinic/pom.xml') =~ '<version>(.+)</version>'
        //             def version = temp[0][1]
        //             env.IMAGE_VERSION = "${version}-${BUILD_NUMBER}"
        //         }
        //     }
        // }

        stage('Build Jar File') {
            steps {
                script {
                    echo 'Building jar file.....'
                    sh 'cd spring-petclinic; ./mvnw clean package -DskipTests'
                    echo 'Jar file built'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image.....'
                    sh "docker build -t ${IMAGE}:${IMAGE_VERSION} -t ${IMAGE}:latest ."
                    echo 'Docker image built'
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                script {
                    echo "Pushing image...."
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh "echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin"
                        sh "docker push ${IMAGE}:${IMAGE_VERSION}"
                        sh "docker push ${IMAGE}:latest"
                    }
                    echo 'Image pushed'    
                }
            }
        }

        stage('Deploy to Petclinic Server') {
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
        

    //     stage('Push pom.xml File') {
    //         steps {
    //             script {
    //                 withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
    //                     sh "git config --global user.name 'jenkins'"
    //                     sh "git config --global user.email 'jenkins@example.com'"
    //                     sh "git remote set-url origin https://${USERNAME}:${PASSWORD}@github.com/Alaa7Hany/petclinic-complete-pipeline.git"
    //                     sh "git add spring-petclinic/pom.xml"
    //                     sh "git commit -m 'Version bump [skip ci]'" // Add the skip message here
    //                     sh "git push origin HEAD:main"
    //                 }
    //             }
    //         }
    //     }        
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
