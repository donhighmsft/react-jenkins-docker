pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    tools {
        nodejs 'node-latest'
    }
     parameters {

        string(name: 'REGISTRY_CRED', defaultValue: 'container-registry', description: '')
        string(name: 'IMAGE_REPO_NAME', defaultValue: 'jenkinsdemoregistry.azurecr.io', description: '')
        string(name: 'IMAGE_NAME', defaultValue: 'react-jenkins/react-frontend', description: '')
        string(name: 'LATEST_BUILD_TAG', defaultValue: 'build-latest', description: '')
        string(name: 'DOCKER_COMPOSE_FILENAME', defaultValue: 'docker-compose.yml', description: '')
        string(name: 'DOCKER_STACK_NAME', defaultValue: 'react_stack', description: '')
        booleanParam(name: 'NPM_RUN_TEST', defaultValue: true, description: '')
        booleanParam(name: 'PUSH_DOCKER_IMAGES', defaultValue: true, description: '')
        booleanParam(name: 'DOCKER_STACK_RM', defaultValue: false, description: 'Remove previous stack.  This is required if you have updated any secrets or configs as these cannot be updated. ')
    }
    stages {
        stage('Welcome Step') {
            steps { 
                echo 'react-jenkins-docker Again'
            }
        }
        stage('npm install'){
            steps {
                sh "npm install"
            }
        }
        stage('npm test'){
            when{
                expression{
                    return params.NPM_RUN_TEST
                }
            }
            steps{
                sh "npm test -- --coverage"	
            }
        }
        stage('npm build'){
            steps{
                sh "npm run build"
            }
        }
        stage('docker build'){
            environment {
                COMMIT_TAG = sh(returnStdout: true, script: 'git rev-parse HEAD').trim().take(7)
                BUILD_IMAGE_REPO_TAG = "${params.IMAGE_REPO_NAME}/${params.IMAGE_NAME}:${env.BUILD_TAG}"
            }
            steps{
                sh "docker build . -t $BUILD_IMAGE_REPO_TAG"
                sh "docker tag $BUILD_IMAGE_REPO_TAG ${params.IMAGE_NAME}:$COMMIT_TAG"
                sh "docker tag $BUILD_IMAGE_REPO_TAG ${params.IMAGE_NAME}:${readJSON(file: 'package.json').version}"
                sh "docker tag $BUILD_IMAGE_REPO_TAG ${params.IMAGE_NAME}:${params.LATEST_BUILD_TAG}"
                sh "docker tag $BUILD_IMAGE_REPO_TAG ${params.IMAGE_NAME}:$BRANCH_NAME-latest"

            }
        }
        stage('docker push image'){
            when{
                expression {
                return params.PUSH_DOCKER_IMAGES
                }
            }
            environment {
                COMMIT_TAG = sh(returnStdout: true, script: 'git rev-parse HEAD').trim().take(7)
                BUILD_IMAGE_REPO_TAG = "${params.IMAGE_REPO_NAME}/${params.IMAGE_NAME}:${env.BUILD_TAG}"
            }
            steps{
                withRegistry(${params.IMAGE_REPO_NAME}, ${params.REGISTRY_CRED}) {
                    sh "docker push $BUILD_IMAGE_REPO_TAG"
                    sh "docker push $BUILD_IMAGE_REPO_TAG ${params.IMAGE_NAME}:$COMMIT_TAG"
                    sh "docker push $BUILD_IMAGE_REPO_TAG ${params.IMAGE_NAME}:${readJSON(file: 'package.json').version}"
                    sh "docker push $BUILD_IMAGE_REPO_TAG ${params.IMAGE_NAME}:${params.LATEST_BUILD_TAG}"
                    sh "docker push $BUILD_IMAGE_REPO_TAG ${params.IMAGE_NAME}:$BRANCH_NAME-latest"    
                }
            }
        }
    }
    post {
        always {
            echo "Finished the Docker Build and Pushed Image to Registry"
            echo "COMMIT TAG Build of the ${params.IMAGE_NAME}:${commitID()}"
            echo "JSON TAG Build of the ${params.IMAGE_NAME}:${readJSON(file: 'package.json').version}"
        }
    }
}

def commitID() {
    sh 'git rev-parse HEAD > .git/commitID'
    def commitID = readFile('.git/commitID').trim().take(7)
    sh 'rm .git/commitID'
    commitID
}