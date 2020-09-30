pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    app = docker.build("mcweenjr/simple-java-maven-app")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_id') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            steps {
                input 'Deploy to Production?'
                milestone(1)
                script {
                  sh 'docker pull mcweenjr/simple-java-maven-app'
                        try {
                            sh 'docker stop mcweenjr/simple-java-maven-app'
                            sh 'docker rm mcweenjr/simple-java-maven-app'
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh 'docker run -p 8280:8080 -p 9990:9990 -d mcweenjr/simple-java-maven-app /opt/jboss/wildfly/bin/standalone.sh -b 0.0.0.0 -bmanagement 0.0.0.0'
                    }
                }
            }
        }
}
