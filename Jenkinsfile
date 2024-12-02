def imageName="bardys/frontend"
def dockerTag=""
def dockerRegistry=""
def registryCredentials="dockerhub"
 
pipeline {
    agent {
        label 'agent'
    }
 
    environment {
        PIP_BREAK_SYSTEM_PACKAGES = 1
        scannerHome = tool 'SonarQube'
    }
 
    stages {
        stage('Get Code') {
            steps {
                //git branch: 'jenkinsfile', url: 'https://github.com/vBardys/Frontend.git'
                checkout scm
            }
        }
 
        stage('Unit tests') {
            steps {
                sh "pip3 install -r requirements.txt"
                sh "python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml"
            }
        }
 
        stage('Sonarqube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
 
        stage('Build application image') {
            steps {
                script {
                  dockerTag = "RC-${env.BUILD_ID}"
                  applicationImage = docker.build("$imageName:$dockerTag")
                }
            }
        }
 
        stage ('Pushing image to docker registry') {
            steps {
                script {
                    docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                        applicationImage.push()
                        applicationImage.push('latest')
                    }
                }
            }
        } 
        stage ('Push to Repo') {
                    steps {
                        dir('ArgoCD') {
                            withCredentials([gitUsernamePassword(credentialsId: 'git', gitToolName: 'Default')]) {
                                git branch: 'main', url: 'https://github.com/vBardys/ArgoCD.git'
                                sh """ cd frontend
                                sed -i "s#$imageName.*#$imageName:$dockerTag#g" frontend.yaml
                                git commit -am "Set new $dockerTag tag."
                                git push origin main
                                """
                            }
                        }
                    }
                }
            
        
    }
 
    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
        success{
            build job: 'app_of_apps', parameters: [ string(name: 'frontendDockerTag', value: "$dockerTag")], wait: false
        }
    }
}
