pipeline {
    agent any

    environment {
        DOCKER_IMAGE_OWNER = 'sun12wook11'
        DOCKER_BUILD_TAG = "v${env.BUILD_NUMBER}"
        DOCKER_TOKEN = credentials('dockerhub')
        GIT_CREDENTIALS = credentials('github_token')
        REPO_URL = 'sun12wook11/project-jenkins.git'
        ARGOCD_REPO_URL = 'sun12wook11/project-argocd.git'
        COMMIT_MESSAGE = 'Update README.md via Jenkins Pipeline'
    }

    stages {
            stage('Clone from SCM') {
                steps {
                    dir('project-jenkins') {
                        git branch: 'master', url: "https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/${REPO_URL}"
                    }
                    dir('project-argocd') {
                        git branch: 'master', url: "https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/${ARGOCD_REPO_URL}"
                   }
            }
        }
    stage('Docker Image Building') {
          steps {
                sh '''
                docker build -t ${DOCKER_IMAGE_OWNER}/prj-frontend:latest ./frontend
                docker build -t ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_BUILD_TAG} ./frontend
                docker build -t ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_BUILD_TAG} ./admin-service
                docker build -t ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_BUILD_TAG} ./visitor-service
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USR', passwordVariable: 'DOCKER_PWD')]) {
                    sh "echo $DOCKER_PWD | docker login -u $DOCKER_USR --password-stdin"
                }
            }
        }

        stage('Docker Image Pushing') {
            steps {
                sh '''
                docker push ${DOCKER_IMAGE_OWNER}/prj-frontend:latest
                docker push ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_BUILD_TAG}
                docker push ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_BUILD_TAG}
                docker push ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_BUILD_TAG}
                '''
            }
        }

        stage('Update ArgoCD values.yaml with Image Tags') {
            steps {
                dir('project-argocd') {
                    sh """
                    sed -i "/${DOCKER_IMAGE_OWNER}\\/prj-frontend/{n;s/tag: \\".*\\"/tag: \\"${DOCKER_BUILD_TAG}\\"/}" deploy-argocd/values.yaml
                    sed -i "/${DOCKER_IMAGE_OWNER}\\/prj-admin/{n;s/tag: \\".*\\"/tag: \\"${DOCKER_BUILD_TAG}\\"/}" deploy-argocd/values.yaml
                    sed -i "/${DOCKER_IMAGE_OWNER}\\/prj-visitor/{n;s/tag: \\".*\\"/tag: \\"${DOCKER_BUILD_TAG}\\"/}" deploy-argocd/values.yaml
                    """
                }
            }
        }

        stage('Commit Changes') {
            steps {
                dir('project-argocd') {
                    script{
                        def changes = sh(script: "git status --porcelain", returnStdout: true).trim()
                        if (changes) {
                            sh '''
                            git config user.name "sun12wook11"
                            git config user.email "sun12wook11@jenkins.com"
                            git add deploy-argocd/values.yaml
                            git commit -m "Update image tags to ${DOCKER_BUILD_TAG}"
                            '''
                        } else {
                            echo "No changes to commit"
                        }
                    }
                }
            }
        }

        stage('Push Changes') {
            steps {
                dir('project-argocd') {
                    withCredentials([usernamePassword(credentialsId: 'github_token', usernameVariable: 'GIT_CREDENTIALS_USR', passwordVariable: 'GIT_CREDENTIALS_PSW')]) {
                        sh '''
                        git push https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/${ARGOCD_REPO_URL} master
                        '''
                    }
                }
            }
        }
    }
}
