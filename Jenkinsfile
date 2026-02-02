pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "mnlove/mnr:${BUILD_NUMBER}"
    }

    tools {
        maven 'maven'
    }

    stages {

        stage('Checkout App Repo') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/maheshchirram/Deployed-on-k8s-with-GitOps.git'
            }
        }

        stage('Build App') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    withCredentials([string(credentialsId: 'sonar', variable: 'SONAR_TOKEN')]) {
                        sh 'mvn sonar:sonar -Dsonar.login=$SONAR_TOKEN'
                    }
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                        docker build -t $DOCKER_IMAGE .
                        docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Checkout Manifest Repo') {
            steps {
                dir('manifests') {
                    git branch: 'main',
                        url: 'https://github.com/maheshchirram/argocd-123.git'
                }
            }
        }

        stage('Update K8s Manifest') {
            steps {
                dir('manifests') {
                    withCredentials([usernamePassword(
                        credentialsId: 'Github',
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_PASSWORD'
                    )]) {
                        sh '''
                            set -e
                            ls -R

                            git config user.email "jenkins@ci.com"
                            git config user.name "Jenkins CI"

                            sed -i "s|image:.*|image: mnlove/mnr:${BUILD_NUMBER}|g" k8s/deployment.yml

                            git add k8s/deployment.yml
                            git commit -m "Update image to mnlove/mnr:${BUILD_NUMBER}" || echo "No changes"
                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/maheshchirram/argocd-123.git main
                        '''
                    }
                }
            }
        }
    }
}
