pipeline {

    agent { label 'gp-agent' }
    
    environment {
        TAG="$env.BRANCH_NAME"
        REGISTRY="266096842478.dkr.ecr.eu-north-1.amazonaws.com/staff-manager-collaborator-ui"        
        AWS_REGION="eu-north-1"
        TAGGED="$REGISTRY:$TAG"
    }

    stages {
        stage('Install dependencies') {
            steps {
                script {
                    sh "npm install"
                }
            }
        }
        stage('Build Angular App') {
            steps {
                script {
                    sh "npm run build"
                }
            }
        }
        stage("Build Docker Image") {
            steps {
                sh "docker build -t staff-manager-collaborator-ui  ."
            }
        }
        stage('Push to Registry') {
            steps {
                script {
                    withEnv(["AWS_DEFAULT_REGION=$AWS_REGION"]) {
                        sh "aws ecr get-login-password | docker login --username AWS --password-stdin $REGISTRY"
                        sh "docker tag staff-manager-collaborator-ui $TAGGED"
                        if (env.BRANCH_NAME == 'production') {
                            def deployInput = input(
                                id: 'deployToProd',
                                message: 'Do you want to push a new production image ?',
                                parameters: [choice(choices: ['Yes', 'No'], description: 'Choose whether to push or not?', name: 'Push image?')],
                                submitter: 'admin'
                            )
                            if (deployInput == 'Yes') {
                                sh "docker push $TAGGED"
                            } else {
                                echo "Image push skipped by user."
                            }
                        } else {
                            sh "docker push $TAGGED"
                        }
                    }
                }
            }
        }
    }
}