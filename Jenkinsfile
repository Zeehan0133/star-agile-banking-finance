pipeline {
    agent { label 'javaslave1' }

    tools {
        maven "maven_3.6.3"
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerlogin')
    }

    stages {
        stage('SCM_Checkout') {
            steps {
                echo 'Perform SCM Checkout'
                git 'https://github.com/Zeehan0133/star-agile-banking-finance.git'
            }
        }

        stage('Application_Build') {
            steps {
                echo 'Perform Maven Build'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
            post {
                failure {
                    sh "echo 'Send mail on failure'"
                    mail to: "zyne0133@gmail.com", from: 'zyne0133@gmail.com', subject: "FAILURE: ${currentBuild.fullDisplayName}", body: "Build failed."
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker version'
                sh "docker build -t zeehan0133/bankapp-zee-app:V${BUILD_NUMBER} ."
                sh 'docker image list'
                //sh "docker tag zeehan0133/bankapp-zee-app:V${BUILD_NUMBER} zeehan0133/bankapp-zee-app:latest"
            }
            post {
                success {
                    sh "echo 'Send mail docker Build Success'"
                    mail to: "zyne0133@gmail.com", from: 'zyne0133@gmail.com', subject: "App Image Created Please validate", body: "App Image Created Please validate - zeehan0133/bankapp-zee-app:V${BUILD_NUMBER}"
                }
                failure {
                    sh "echo 'Send mail docker Build failure'"
                    mail to: "zyne0133@gmail.com", from: 'zyne0133@gmail.com', subject: "FAILURE: ${currentBuild.fullDisplayName}", body: "Image Build failed."
                }
            }
        }

        stage('Approve - push Image to Docker Hub') {
            steps {
                //----------------send an approval prompt-------------
                script {
                    env.APPROVED_DEPLOY = input message: 'User input required Choose "Yes" | "Abort"'
                }
                //-----------------end approval prompt------------
            }
        }

        stage('Login to Docker Hub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Publish to Docker Registry') {
            steps {
                sh "docker push zeehan0133/bankapp-zee-app:V${BUILD_NUMBER}"
            }
        }

        stage('Approve - Deployment') {
            steps {
                //----------------send an approval prompt-------------
                script {
                    env.APPROVED_DEPLOY = input message: 'User input required Choose "Yes" | "Abort"'
                }
                //-----------------end approval prompt------------
            }
        }

        stage('Deploy to Kubernetes Cluster') {
            steps {
                script {
                    sshPublisher(publishers: [sshPublisherDesc(configName: 'Kubernates', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'kubectl apply -f k8smvndeployment.yaml', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '.', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '*.yaml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
                }
            }
        }
    }
}
