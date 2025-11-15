
    def job = "${env.JOB_NAME}".tokenize('/')
    print job
    def appName = job[0]
    print appName
    def tag = "v2025"


    pipeline{
    agent any
    tools {
        maven "M3"
    }
    environment {
        DOCKER_REPO = 'ranjith413'
        IMAGE_NAME = "${appName}"
        IMAGE_TAG = "${tag}-${env.BUILD_ID}"
    }
    stages {
        stage("checkout") {
            steps {
                sh 'ls -ltr'
                echo 'hello devops!'
            }

        }

        stage("maven-build"){
            agent {
                docker { 
                    image 'maven:3.9-eclipse-temurin-17-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                    reuseNode true
                    }
            }
            steps {
                sh 'mvn clean package'
            }
        }

    stage("Image-build"){
        steps{
            sh 'docker build -t ${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG} .'
        }
    }
    stage("Publish Image"){
        steps {
            script{
                withCredentials([usernamePassword(credentialsId: 'docker-creds', passwordVariable: 'DOCKER_PASSWD', usernameVariable: 'DOCKER_UNAME')]) {
                    sh '''
                    echo ${DOCKER_PASSWD} | docker login -u ${DOCKER_UNAME} --password-stdin
                    docker push ${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                    docker logout
                    '''
                }
            }
        }
    }
    stage("Update Image in manifests"){
        steps{
            withCredentials([usernamePassword(credentialsId: 'github-token', passwordVariable: 'GIT_TOKEN', usernameVariable: 'GIT_USER')]){
            script{
                sh '''
                cd helm-charts/
                BRANCH="release/${IMAGE_TAG}"
                REPO_OWNER="ranjith-duvasi"
                REPO_NAME="${IMAGE_NAME}"
                COMMIT_MESSAGE="release: update image tag ${BRANCH}"
                git config --global user.email "dranjith956@gmail.com"
                git config --global user.name "ranjith-duvasi"
                git checkout -b "${BRANCH}"
                sed -i "s/^\\([[:space:]]*tag:\\).*/\\1 ${IMAGE_TAG}/" values.yaml
                git add values.yaml
                git commit -m "${COMMIT_MESSAGE}"
                git push https://${GIT_USER}:${GIT_TOKEN}@github.com/${REPO_OWNER}/${IMAGE_NAME}.git "${BRANCH}"

                #create a PR
                response=$(curl -s -w "\\n%{http_code}" -H "Authorization: token ${GIT_TOKEN}" \
                -H "Accept: application/vnd.github+json" -H "Content-Type: application/json" \
                -d @- "https://api.github.com/repos/${REPO_OWNER}/${REPO_NAME}/pulls" <<EOF
                {
                    "title": "${COMMIT_MESSAGE}",
                    "head": "${BRANCH}",
                    "base": "main",
                    "body": "Auto PR by Jenkins"
                }
                EOF
                )
                '''

            }
        }
        }

        
    }

    }
    post {
        always {
            cleanWs()
        }
    }
}
