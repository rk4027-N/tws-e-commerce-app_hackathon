@Library('Shared') _

pipeline {
    agent any
    
    environment {
        AWS_REGION = "us-east-1"
        AWS_ACCOUNT_ID = "984912522187"

        DOCKER_IMAGE_NAME = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/easyshop-app"
        DOCKER_MIGRATION_IMAGE_NAME = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/easyshop-migration"

        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"

        GIT_BRANCH = "master"
    }
    
    stages {

        stage('Cleanup Workspace') {
            steps {
                script {
                    clean_ws()
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    clone("https://github.com/rk4027-N/tws-e-commerce-app_hackathon.git","master")
                }
            }
        }

        stage('Login to ECR') {
            steps {
                script {
                    sh """
                    aws ecr get-login-password --region ${AWS_REGION} \
                    | docker login \
                    --username AWS \
                    --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    """
                }
            }
        }

        stage('Build Docker Images') {
            parallel {

                stage('Build Main App Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'Dockerfile',
                                context: '.'
                            )
                        }
                    }
                }

                stage('Build Migration Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'scripts/Dockerfile.migration',
                                context: '.'
                            )
                        }
                    }
                }

            }
        }

        stage('Run Unit Tests') {
            steps {
                script {
                    run_tests()
                }
            }
        }

        stage('Security Scan with Trivy') {
            steps {
                script {
                    trivy_scan()
                }
            }
        }

        stage('Push Docker Images') {
            parallel {

                stage('Push Main App Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG
                            )
                        }
                    }
                }

                stage('Push Migration Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG
                            )
                        }
                    }
                }

            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    update_k8s_manifests(
                        imageTag: env.DOCKER_IMAGE_TAG,
                        manifestsPath: 'kubernetes',
                        gitCredentials: 'Github',
                        gitUserName: 'rk4027-N',
                        gitUserEmail: 'gudalarajkumar4444@gmail.com'
                    )
                }
            }
        }
    }
}
