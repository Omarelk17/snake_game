pipeline {
    agent none
    stages {
        stage('CLONE GIT REPOSITORY') {
            agent {
                label 'ubuntu-Appserver-2140'
            }
            steps {
                checkout scm
            }
        }

        stage('INSTALL DEPENDENCIES') {
            agent {
                label 'ubuntu-Appserver-2140'
            }
            steps {
                sh 'npm install'
            }
        }

        stage('SCA-SAST-SNYK-TEST') {
            agent any
            steps {
                script {
                    snykSecurity(
                        snykInstallation: 'Snyk',
                        snykTokenId: 'synk_api',
                        severity: 'critical'
                    )
                }
            }
        }

        stage('SonarQube Analysis') {
            agent {
                label 'ubuntu-Appserver-2140'
            }
            steps {
                script {
                    def scannerHome = tool 'sonarqube'
                    withSonarQubeEnv('sonarqube') {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=snake_game \
                            -Dsonar.sources=."
                    }
                }
            }
        }

        stage('BUILD-AND-TAG') {
            agent {
                label 'ubuntu-Appserver-2140'
            }
            steps {
                script {
                    def app = docker.build("omarelk18/snake_game")
                    app.tag("latest")
                }
            }
        }

        stage('POST-TO-DOCKERHUB') {    
            agent {
                label 'ubuntu-Appserver-2140'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_credentials') {
                        def app = docker.image("omarelk18/snake_game")
                        app.push("latest")
                    }
                }
            }
        }

        stage('DEPLOYMENT') {    
            agent {
                label 'ubuntu-Appserver-2140'
            }
            steps {
                sh "docker-compose down"
                sh "docker-compose up -d"   
            }
        }
    }
}
