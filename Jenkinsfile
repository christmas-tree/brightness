pipeline {
    agent any
    tools {
        maven 'maven'
        jdk 'jdk8'
    }
    triggers {
        pollSCM('* * * * *')
    }

    stages {
        stage('Checkout code') {
            steps {
                checkout scm
            }
        }

        stage ('Compile') {
            steps {
                sh 'mvn -B clean package -Dmaven.test.skip=true'
            }
            post {
                always {
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }

        stage ('Test') {
            steps {
                sh 'mvn -B test -Dmaven.test.failure.ignore=true'
            }
        }

        stage ('Sonar') {
            steps {
                sh 'mvn sonar:sonar'
            }
        }

        stage ('Deploy snapshot') {
            when {
                not {
                    branch('master')
                }
            }
            parallel {
                stage ('Deploy snapshot') {
                    steps {
                        sh "mvn deploy -DaltDeploymentRepository=$DEPLOYMENT_REPOSITORY"
                    }
                }
                stage ('Docker snapshot') {
                    steps {
                        script {
                            docker.build "christmas-tree-brightness:$BUILD_NUMBER"
                        }
                    }
                }
            }
        }
        stage ('Deploy master') {
            when {
                branch('master')
            }
            parallel {
                stage ('Deploy release') {
                    steps {
                        sh 'mvn deploy -Psonatype-oss-release'
                    }
                }
                stage ('Docker snapshot') {
                    steps {
                        script {
                            docker.build "$DOCKER_REPO/christmas-tree-brightness:latest"
                            docker.push "$DOCKER_REPO/christmas-tree-brightness:latest"
                        }
                    }
                }
            }
        }
    }
}