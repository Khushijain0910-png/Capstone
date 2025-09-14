pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node23'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'   // Sonar scanner tool in Jenkins
        SONAR_TOKEN = credentials('Sonar-token') // SonarQube token
        REPO_NAME = 'khushijain0910/capstone-project'
        IMAGE_NAME = 'bms-app'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Khushijain0910-png/Capstone.git'
                sh 'ls -la'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=Capstone_Project \
                        -Dsonar.projectName=Capstone_Project \
                        -Dsonar.sources=bookmyshow-app \
                        -Dsonar.host.url=http://54.153.76.43:9000/ \
                        -Dsonar.login=$SONAR_TOKEN
                    """
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('bookmyshow-app') {
                    sh '''
                        if [ -f package.json ]; then
                            rm -rf node_modules package-lock.json
                            npm install
                        else
                            echo "Error: package.json not found!"
                            exit 1
                        fi
                    '''
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                dir('bookmyshow-app') {
                    script {
                        sh "docker build -t ${REPO_NAME}:${env.BUILD_NUMBER} ."
                        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                            sh "docker push ${REPO_NAME}:${env.BUILD_NUMBER}"
                        }
                    }
                }
            }
        }

        stage('Deploy to Container') {
            steps {
                sh '''
                    docker rm -f bms-app || true
                    docker run -d --name bms-app -p 3000:3000 khushijain0910/capstone-project:${BUILD_NUMBER}
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Email notifications are disabled."
        }
    }
}


        

       

