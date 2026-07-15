pipeline {
    agent any

    tools {
        maven 'Maven-3'
        jdk 'JDK-21'
    }

    environment {
        APP_NAME = 'jenkins-cicd-demo'
        IMAGE_NAME = 'your-dockerhub-username/jenkins-cicd-demo'
        IMAGE_TAG = "${BUILD_NUMBER}"
        NEXUS_URL = 'http://your-nexus-server:8081'
        NEXUS_REPO = 'maven-releases'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/your-username/jenkins-cicd-demo.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Run Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=jenkins-cicd-demo \
                    -Dsonar.projectName=jenkins-cicd-demo
                    '''
                }
            }
        }

        stage('Upload Artifact to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-credentials',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh '''
                    mvn deploy:deploy-file \
                    -DgroupId=com.devopsher \
                    -DartifactId=jenkins-cicd-demo \
                    -Dversion=1.0.${BUILD_NUMBER} \
                    -Dpackaging=jar \
                    -Dfile=target/jenkins-cicd-demo-1.0.0.jar \
                    -DrepositoryId=nexus \
                    -Durl=${NEXUS_URL}/repository/${NEXUS_REPO}/ \
                    -Dusername=${NEXUS_USER} \
                    -Dpassword=${NEXUS_PASS}
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                docker stop jenkins-cicd-demo || true
                docker rm jenkins-cicd-demo || true
                docker run -d --name jenkins-cicd-demo -p 8080:8080 $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check Jenkins console logs.'
        }
    }
}
