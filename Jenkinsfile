pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh 'chmod +x gradlew'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/reactApp'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'main'
            }
            steps {
                script {
                    app = docker.build("kiloabachie/react-app")
                    app.inside {
                        sh 'echo $(curl localhost:1233)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'main'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
         stage('DeployToStaging') {
            when {
                branch 'main'
            }
            steps {
                    script {
                        

// Pull the Docker image
sh "docker pull kiloabachie/react-app:${env.BUILD_NUMBER}"

// Get IDs of all running containers
            def doc_containers = sh(returnStdout: true, script: 'docker container ls -aq').trim()

// Check if any containers are running
            if (doc_containers) {
    // Stop and remove all running containers
                    sh "docker stop $doc_containers"
                    sh "docker rm $doc_containers"
}
                        sh "docker run --restart always --name react-app -p 1233:80 -d kiloabachie/react-app:${env.BUILD_NUMBER}"
                    }
            }
        }
        
        stage("Check HTTP Response") {
            steps {
                script {
                    final String url = "http://localhost:1233"
                    
                    final String response = sh(script: "curl -o /dev/null -s -w '%{http_code}\\n' $url", returnStdout: true).trim()
                    
                    if (response == "200") {
                        echo response
                        println "Successful Response Code" 
                    } else {
                        echo response
                        println "Error Response Code" 
                    }

                }
            }
        }
        
        stage('DeployToProduction') {
            when {
                branch 'main'
            }
            steps {
                input 'Does the staging environment look OK? Did You get 200 response?'
                 milestone(1)
                    script {
                        sh "docker pull kiloabachie/react-app:${env.BUILD_NUMBER}"
                        def container = sh(returnStdout: true, script: 'docker ps -a --format "{{.Names}}" | grep react-app')
                         if (container) {
                           sh "docker stop ${container}"
                           sh "docker rm ${container}"
                         }
                        sh "docker run --restart always --name react-app -p 1233:80 -d kiloabachie/react-app:${env.BUILD_NUMBER}"
                    }
            }
        }
    }
}
