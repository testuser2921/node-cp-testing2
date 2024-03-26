pipeline {
    agent any

    tools {
        nodejs 'NodeJS'
    }

    environment {
                PROJECT_ID = "tech-rnd-project"
                CLUSTER_NAME = "cluster"
                LOCATION = "us-central1-a"
                CREDENTIALS_ID = "kubernetes"
    }

    stages {
        stage('Scm Checkout') {
            steps {
                    checkout scm
            }
        }
        stage('Build') {
            steps {
                echo 'building the software'
                sh 'npm install --force'
            }
        }
        stage('Test') {
            steps {
                 //add steps here
                echo 'testing'
            }
        }


        stage('Build Docker Image') {
            steps {
                sh 'whoami'
                sh 'sudo chmod 777 /var/run/docker.sock'
                sh 'sudo apt update'
                sh 'sudo apt install software-properties-common -y'
                sh 'sudo add-apt-repository ppa:cncf-buildpacks/pack-cli'
                sh 'sudo  apt-get update'
                sh 'sudo apt-get install pack-cli'
                sh 'pack build java-postgress --builder gcr.io/buildpacks/google-22/builder@sha256:ffa092c09ffb147b2ce9658eb6590aa9af2caffb2d513ab8546bb510b74e0225'
            }
        }

        // stage('Image Scan') {
        //     steps {
        //         //add steps here
        //         echo 'image scanning'
        //         sh 'trivy image --no-progress --exit-code 1 --severity CRITICAL java-postgress'
        //     }
        // }

        stage('Push Docker Image') {
            steps {
                script {
                    echo 'Push Docker Image'
                    sh 'docker tag java-postgress us-central1-docker.pkg.dev/tech-rnd-project/container-node/nodejs'
                    sh 'gcloud auth configure-docker us-central1-docker.pkg.dev'
                    sh 'docker push us-central1-docker.pkg.dev/tech-rnd-project/container-node/nodejs'
                    // sh 'curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl'
                    // sh 'chmod +x kubectl'
                    // sh "sudo mv kubectl \$(which kubectl)"
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                echo 'Deployment started ...'
                sh 'ls -ltr'
                sh 'pwd'
                echo 'Start deployment of deployment.yaml'
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'k8', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
                    echo 'Deployment Finished ...'
                sh '''
                '''
            }
        }
    }
    post {
        always {
            emailext to: 'vinod.bangera@niveussolutions.com',
            subject: "jenkins build:${currentBuild.currentResult}: ${env.JOB_NAME}",
            body: "${currentBuild.currentResult}: Job ${env.JOB_NAME}\nMore Info can be found in the attached log",
            attachLog: true
        }
    }
}
