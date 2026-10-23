```groovy
pipeline {
    agent any

    tools {
        maven 'maven3.9.12'
        jdk 'java17'
    }

    environment {
        APP_NAME         = 'country-chicken-backend'
        NEXUS_MAVEN_URL  = 'local-nexus:8081'
        NEXUS_DOCKER_URL = 'localhost:8082'
        MAVEN_REPO       = 'maven-releases'
        DOCKER_REPO      = 'docker-releases'
        GROUP_ID         = 'com.countrychicken'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/munaffdevops/country-chicken-backend-ci.git'
            }
        }

        stage('Read Version') {
            steps {
                script {
                    env.VERSION = sh(
                        script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout',
                        returnStdout: true
                    ).trim()

                    env.JAR_NAME = "${env.APP_NAME}-${env.VERSION}.jar"

                    echo "Project version: ${env.VERSION}"
                }
            }
        }

        stage('Build and Test') {
            steps {
                sh 'mvn clean verify'
            }
        }

        stage('Upload JAR to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'local-nexus:8081',
                    groupId: "${env.GROUP_ID}",
                    version: "${env.VERSION}",
                    repository: "${env.MAVEN_REPO}",
                    credentialsId: 'nexus-credentials',
                    artifacts: [[
                        artifactId: "${env.APP_NAME}",
                        classifier: '',
                        file: "target/${env.JAR_NAME}",
                        type: 'jar'
                    ]]
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${NEXUS_DOCKER_URL}/${DOCKER_REPO}/${APP_NAME}:${VERSION} .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-credentials',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh """
                        echo "\$NEXUS_PASS" | docker login ${NEXUS_DOCKER_URL} \
                            -u "\$NEXUS_USER" --password-stdin

                        docker push ${NEXUS_DOCKER_URL}/${DOCKER_REPO}/${APP_NAME}:${VERSION}

                        docker logout ${NEXUS_DOCKER_URL}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Build and publish successful.'
        }
        failure {
            echo 'Build failed. Check the stage logs.'
        }
    }
}
```
